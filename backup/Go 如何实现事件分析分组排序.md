事件分析过程中，会对分组进行排序，有如下的排序规则，

1. 按第一个指标值的正序排列
2. 按第一个指标值的倒序排列
3. 按分组值的正序排列
4. 按分组值的倒序排列

分组是 `[][]string` 类型，值是 `float64` 类型，

下面记录下主要的排序代码

```go
const (
	PropertyTypeInteger              = "integer"                // 整数类型
	PropertyTypeFloat                = "float"                  // 浮点类型，对应 float64 或 double
	PropertyTypeNumber               = "number"                 // 浮点类型，对应 float64 或 double
)

type EventModelQueryGroupBy struct {
	Name              string    `json:"name" example:"_time"`
	Type              string    `json:"type" example:"timestamp"`
	Title             string    `json:"title" example:"事件发生时间"`
	TableType         string    `json:"table_type" example:"event"`
	MetaType          string    `json:"meta_type" example:"custom"`
}

// 根据分组值排序
func sortUnionGroupsByValue(unionGroups [][]string, unionGroupsValueMap map[string]float64, unionGroupsJoinSymbol string, sortByAsc bool) [][]string {
	newUnionGroups := make([][]string, len(unionGroups))
	copy(newUnionGroups, unionGroups)
	sort.Slice(newUnionGroups, func(i, j int) bool {
		sortValue := false
		if sortByAsc {
			sortValue = unionGroupsValueMap[strings.Join(newUnionGroups[i], unionGroupsJoinSymbol)] < unionGroupsValueMap[strings.Join(newUnionGroups[j], unionGroupsJoinSymbol)]
		} else {
			sortValue = unionGroupsValueMap[strings.Join(newUnionGroups[i], unionGroupsJoinSymbol)] > unionGroupsValueMap[strings.Join(newUnionGroups[j], unionGroupsJoinSymbol)]
		}
		return sortValue
	})
	return newUnionGroups
}

// 根据分组名称排序
func sortUnionGroupsByGroup(unionGroups [][]string, unionGroupBy []*EventModelQueryGroupBy, sortByAsc bool) [][]string {
	newUnionGroups := make([][]string, len(unionGroups))
	copy(newUnionGroups, unionGroups)
	sort.Slice(newUnionGroups, func(i, j int) bool {
		sortValue := false
		for index := range unionGroupBy {
			if newUnionGroups[i][index] != newUnionGroups[j][index] {
				switch unionGroupBy[index].Type {
				case PropertyTypeInteger, PropertyTypeNumber:
					if sortByAsc {
						sortValue = sortValue || parseNumberRangeSortValue(newUnionGroups[i][index]) < parseNumberRangeSortValue(newUnionGroups[j][index])
					} else {
						sortValue = sortValue || parseNumberRangeSortValue(newUnionGroups[i][index]) > parseNumberRangeSortValue(newUnionGroups[j][index])
					}
				default:
					if sortByAsc {
						sortValue = sortValue || newUnionGroups[i][index] < newUnionGroups[j][index]
					} else {
						sortValue = sortValue || newUnionGroups[i][index] > newUnionGroups[j][index]
					}
				}
				break
			}
		}
		return sortValue
	})
	return newUnionGroups
}

// 对数值类型分组值拆分得到排序值 [-∞, 100), [-1, 2), 100, (null)
func parseNumberRangeSortValue(valueStr string) float64 {
	firstPartValue := strings.SplitN(valueStr, ",", 2)[0]
	firstPartValue = strings.TrimLeft(firstPartValue, "[")
	switch firstPartValue {
	case "-∞":
		return math.Inf(-1)
	case "(null)":
		return math.NaN()
	default:
		value, err := strconv.ParseFloat(firstPartValue, 64)
		if err != nil {
			return math.Inf(-1)
		}
		return value
	}
}
```