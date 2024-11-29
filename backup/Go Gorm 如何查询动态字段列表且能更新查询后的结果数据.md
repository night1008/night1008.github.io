### 背景

最近广告业务上的一些报表查询接口，需要根据前端指定的字段列表返回查询聚合结果，其中某些字段不方便在数据库中直接查询得到，需要经过服务端二次计算，才能得到最终结果。

广告报表的指标示例如下所示，有些要经过包含变量的数学四则运算，

```yaml
- name: view_count
  title: 曝光次数
  type: integer
  description: 广告被展示给用户的次数
- name: view_user_count
  title: 曝光人数
  type: integer
  description: 产生广告曝光的独立用户数
- name: avg_view_per_user
  title: 人均曝光次数
  type: float
  description: 人均产生的广告曝光次数，计算公式是：曝光次数/曝光人数
  expression: view_count / view_user_count
```

### 实现思路

1. 通过待查询的字段名称列表得到指标对象 fields => reportFields
2. 筛选基础指标对象构造查询字段列表(不包含表达式的指标对象，因为其需要经过四则运算，后续在服务端进行二次处理，可能需要考虑包含自身的情况)
3. 构造查询
4. 二次处理包含表达式的指标对象

```go
type AdReportField struct {
	AppID          uint64   `gorm:"uniqueIndex:idx_ad_report_fields_name"` // 0 标识所有应用共有
	Name           string   `gorm:"uniqueIndex:idx_ad_report_fields_name"` // 字段名称
	Title          string   // 字段标题
	Type           string   // 字段类型
	Description    string   // 字段描述
	IsPercent      bool     // 是否百分比
	Expression     string   // 数学表达式
	ScaleDownValue *float64 // 缩小比例
}

// 转换待查询的字段列表为原始的报表字段
func ConvertStringFieldsToAdReportFields(fieldNames []string, allReportFields map[string]*AdReportField) ([]*AdReportField, error) {
	reportFields := make([]*model.AdReportField, 0, len(allReportFields))
	existedReportFieldsMap := make(map[string]struct{}, len(allReportFields))
	for _, fieldName := range fieldNames {
		reportField, ok := allReportFields[fieldName]
		if !ok {
			continue
		}
		if _, ok := existedReportFieldsMap[fieldName]; ok {
			continue
		}
		reportFields = append(reportFields, reportField)
		existedReportFieldsMap[reportField.Name] = struct{}{}
		if reportField.Expression != "" {
			tokens, err := reportField.ParseMathTokens()
			if err != nil {
				return nil, err
			}
			for _, token := range tokens {
				switch token.Type {
				case mathtoken.TypeVariable:
					tokenField, ok := allReportFields[token.Value]
					if !ok {
						continue
					}
					if _, ok := existedReportFieldsMap[tokenField.Name]; ok {
						continue
					}
					reportFields = append(reportFields, tokenField)
					existedReportFieldsMap[tokenField.Name] = struct{}{}
				}
			}
		}
	}
	return reportFields, nil
}
```

---

### 思路一：通过反射机制创建动态结构体

```go
func queryByReflect(fields []string) (interface{}, error) {
  var groupBy []string
  allSelectFields := append(groupBy, fields...)
  reportFields, err := ConvertStringFieldsToAdReportFields(allSelectFields, adReportFieldMap)
	if err != nil {
		return nil, err
	}
  
  var selectColumnNames []string
  for _, reportField := range reportFields {
		if reportField.Expression != "" {
			continue
		}
		switch reportField.Type {
		case PropertyTypeInteger, PropertyTypeFloat:
			if reportField.ScaleDownValue != nil {
				selectColumnNames = append(selectColumnNames, fmt.Sprintf("SUM((data->>'%s')::float) / %f AS %s", reportField.Name, *reportField.ScaleDownValue, reportField.Name))
			} else {
				selectColumnNames = append(selectColumnNames, fmt.Sprintf("SUM((data->>'%s')::float) AS %s", reportField.Name, reportField.Name))
			}
		}
	}

  rowStruct, err := getReportFieldRowStruct(reportFields)
	if err != nil {
		return nil, err
	}

	items := reflect.New(reflect.SliceOf(rowStruct)).Interface()
	if err := s.DB.Model(&AdTencentReport{}).
		Select(selectColumnNames).
		Group("date").
		Find(items).Error; err != nil {
		return nil, err
	}

  itemValues := reflect.ValueOf(reflect.ValueOf(items).Elem().Interface())
	for _, reportField := range reportFields {
		reportFieldTitleName := getAdReportFieldTitleName(reportField.Name)
		if reportField.Expression != "" {
			tokens, err := reportField.GetMathTokens()
			if err != nil {
				return nil, err
			}
			expression, err := govaluate.NewEvaluableExpression(reportField.Expression)
			if err != nil {
				return nil, err
			}
			for i := 0; i < itemValues.Len(); i++ {
				parameters := make(map[string]interface{}, 8)
				for _, token := range tokens {
					switch token.Type {
					case mathtoken.TypeVariable:
						tokenValue := itemValues.Index(i).FieldByName(getAdReportFieldTitleName(token.Value))
						var value float64
						if !tokenValue.IsNil() {
							value = tokenValue.Elem().Float()
						}
						parameters[token.Value] = value
					}
				}
				expressionResult, err := expression.Evaluate(parameters)
				if err != nil {
					return nil, err
				}

				switch expressionValue := expressionResult.(type) {
				case float64:
					if !math.IsInf(expressionValue, 0) && !math.IsNaN(expressionValue) {
						percentNum := 1.0
						if reportField.IsPercent {
							percentNum = 100.0
						}
						value := expressionValue * percentNum
						itemValues.Index(i).FieldByName(reportFieldTitleName).Set(reflect.ValueOf(&value))
					}
				}
			}
		}
	}

	return itemValues.Interface(), nil
}

// 获取报表字段动态反射结构体
func getReportFieldRowStruct(reportFields []*AdReportField) (reflect.Type, error) {
	var structFields []reflect.StructField
	for _, field := range reportFields {
		name := getAdReportFieldTitleName(field.Name)
		switch field.Type {
		case PropertyTypeString:
			_type := ""
			structFields = append(structFields, reflect.StructField{
				Name: name,
				Type: reflect.TypeOf(&_type),
				Tag:  reflect.StructTag(fmt.Sprintf(`json:"%s" gorm:"%s"`, field.Name, field.Name)),
			})
		case PropertyTypeInteger, PropertyTypeFloat:
			_type := float64(0)
			structFields = append(structFields, reflect.StructField{
				Name: name,
				Type: reflect.TypeOf(&_type),
				Tag:  reflect.StructTag(fmt.Sprintf(`json:"%s" gorm:"%s"`, field.Name, field.Name)),
			})
		default:
			return nil, fmt.Errorf("report field name %s type %s not found", field.Name, field.Type)
		}
	}
	return reflect.StructOf(structFields), nil
}

// 字段名称下划线转驼峰
func getAdReportFieldTitleName(name string) string {
	parts := strings.Split(name, "_")
	titleParts := make([]string, 0, len(parts))
	for _, part := range parts {
		titleParts = append(titleParts, cases.Title(language.English).String(part))
	}
	return strings.Join(titleParts, "")
}
```

---

### 思路二：通过字典方式存储动态数据结构

```go
func queryByMap(fields []string) (interface{}, error) {
  var groupBy []string
  allSelectFields := append(groupBy, fields...)
  reportFields, err := ConvertStringFieldsToBaseAdReportFields(allSelectFields, adReportFieldMap)
	if err != nil {
		return nil, err
	}

  var selectColumnNames []string
  for _, reportField := range reportFields {
		if reportField.Expression != "" {
			continue
		}
		switch reportField.Type {
		case PropertyTypeInteger, PropertyTypeFloat:
			if reportField.ScaleDownValue != nil {
				selectColumnNames = append(selectColumnNames, fmt.Sprintf("SUM((data->>'%s')::float) / %f AS %s", reportField.Name, *reportField.ScaleDownValue, reportField.Name))
			} else {
				selectColumnNames = append(selectColumnNames, fmt.Sprintf("SUM((data->>'%s')::float) AS %s", reportField.Name, reportField.Name))
			}
		}
	}

  var items []map[string]interface{}
	if err := db.Table("ad_reports").
		Select(selectColumnNames).
		Group("date").
		Find(&items).Error; err != nil {
		return nil, err
	}

  for _, reportField := range reportFields {
		if reportField.Expression != "" {
			tokens, err := reportField.ParseMathTokens()
			if err != nil {
				return nil, err
			}
			expression, err := govaluate.NewEvaluableExpression(reportField.Expression)
			if err != nil {
				return nil, err
			}
			for _, item := range items {
				parameters := make(map[string]interface{}, 8)
				for _, token := range tokens {
					switch token.Type {
					case mathtoken.TypeVariable:
						tokenValue := item[token.Value]
						var value float64
						if tokenValue != nil {
							value = tokenValue.(float64)
						}
						parameters[token.Value] = value
					}
				}
				expressionResult, err := expression.Evaluate(parameters)
				if err != nil {
					return nil, err
				}

				var hasValue bool
				switch expressionValue := expressionResult.(type) {
				case float64:
					if !math.IsInf(expressionValue, 0) && !math.IsNaN(expressionValue) {
						percentNum := 1.0
						if reportField.IsPercent {
							percentNum = 100.0
						}
						value := expressionValue * percentNum
						item[reportField.Name] = &value
						hasValue = true
					}
				}
				if !hasValue {
					item[reportField.Name] = nil
				}
			}
		}
  }
  return items, nil
}
```