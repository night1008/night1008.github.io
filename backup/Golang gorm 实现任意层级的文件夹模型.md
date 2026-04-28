记录一下如何使用 gorm 实现任意层级的文件夹模型，代码如下


```go
type AdFolder struct {
	ID        uint64 `gorm:"primarykey"`           // 数据库ID
	CreatedAt int64  `gorm:"autoCreateTime:milli"` // 记录创建时间，单位 milli
	UpdatedAt int64  `gorm:"autoUpdateTime:milli"` // 记录更新时间，单位 milli

	AppID       uint64 `gorm:"index:idx_ad_folders_type"` // 应用ID
	Type        string `gorm:"index:idx_ad_folders_type"` // 分类类型
	Title       string // 标题
	Description string // 说明
	ParentID    uint64 `gorm:"index"` // 父文件夹ID
	RootID      uint64 `gorm:"index"` // 根文件夹ID

	CreatorID    uint64 // 创建用户ID
	LastEditorID uint64 // 最后编辑用户ID

	Parent      *AdFolder   `gorm:"foreignKey:ParentID"` // 父节点
	Children    []*AdFolder `gorm:"foreignKey:ParentID"` // 直系子节点
	Root        *AdFolder   `gorm:"foreignKey:RootID"`   // 根节点
	Descendants []*AdFolder `gorm:"foreignKey:RootID"`   // 根系子节点
}


// 获取根文件夹ID
func (a *AdFolder) GetFolderRootID() uint64 {
	var rootID uint64
	if a.RootID != 0 {
		rootID = a.RootID
	} else if a.ID != 0 {
		rootID = a.ID
	}
	return rootID
}

// 获取相同根节点的文件夹列表
func (a *AdFolder) GetSameRootAdFolders(tx *gorm.DB) ([]*AdFolder, error) {
	var adFolders []*AdFolder
	query := tx.Model(&AdFolder{}).
		Where("app_id = ?", a.AppID).
		Where("type = ?", a.Type)
	if a.RootID == 0 {
		query = query.Where("(id = ? OR root_id = ?)", a.ID, a.ID)
	} else {
		query = query.Where("(id = ? OR root_id = ?)", a.RootID, a.RootID)
	}
	if err := query.Order("id DESC").
		Find(&adFolders).Error; err != nil {
		return nil, err
	}
	return adFolders, nil
}

// 构造文件夹父节点数组
func (a *AdFolder) GetAdFolderParents(tx *gorm.DB) ([]*AdFolder, error) {
	adFolders, err := a.GetSameRootAdFolders(tx)
	if err != nil {
		return nil, err
	}
	adFoldersMap := make(map[uint64]*AdFolder, len(adFolders))
	parentAdFoldersMap := make(map[uint64]uint64, len(adFolders)) // children : parent
	for _, folder := range adFolders {
		adFoldersMap[folder.ID] = folder
		if folder.ParentID != 0 {
			parentAdFoldersMap[folder.ID] = folder.ParentID
		}
	}

	var parents []*AdFolder
	currentID := a.ID
	for {
		parentID, exists := parentAdFoldersMap[currentID]
		if !exists {
			break
		}

		parent, ok := adFoldersMap[parentID]
		if !ok {
			break
		}

		parents = append([]*AdFolder{parent}, parents...)
		currentID = parentID
	}
	return parents, nil
}

// 构造文件夹树形结构
func BuildAdFolderTree(adFolders []*AdFolder) []*AdFolder {
	adFoldersMap := make(map[uint64]*AdFolder, len(adFolders))
	for _, adFolder := range adFolders {
		adFoldersMap[adFolder.ID] = adFolder
	}

	var roots []*AdFolder
	for _, adFolder := range adFolders {
		if adFolder.ParentID == 0 {
			roots = append(roots, adFolder)
		} else {
			if parent, ok := adFoldersMap[adFolder.ParentID]; ok {
				parent.Children = append(parent.Children, adFolder)
				adFoldersMap[adFolder.ParentID] = parent
			}
		}
	}
	return roots
}

// 寻找所有后代节点
func (a *AdFolder) GetAdFolderDescendants(tx *gorm.DB) ([]*AdFolder, error) {
	adFolders, err := a.GetSameRootAdFolders(tx)
	if err != nil {
		return nil, err
	}
	adFoldersMap := make(map[uint64]*AdFolder)
	childrenMap := make(map[uint64][]uint64)

	for _, adFolder := range adFolders {
		adFoldersMap[adFolder.ID] = adFolder
		if adFolder.ParentID != 0 {
			childrenMap[adFolder.ParentID] = append(childrenMap[adFolder.ParentID], adFolder.ID)
		}
	}

	var children []*AdFolder
	queue := []uint64{a.ID}
	for len(queue) > 0 {
		currentID := queue[0]
		queue = queue[1:]
		for _, childID := range childrenMap[currentID] {
			if child, exists := adFoldersMap[childID]; exists {
				children = append(children, child)
				queue = append(queue, childID)
			}
		}
	}
	return children, nil
}
```