###ExpandableRecyclerView

> 重要的类

- ExpandableRecyclerAdapterHelper

```
List<Object> mHelperItemList;

public List<Object> generateHelperItemList(List<Object> itemList){
	ArrayList<Object> parentWrapperList = new ArrayList<>();
	for(int i = 0;i<itemList.size();i++){
		if (itemList.get(i) instanceof ParentObject) {
                ParentWrapper parentWrapper = new ParentWrapper(itemList.get(i), sCurrentId);
                sCurrentId++;
                parentWrapperList.add(parentWrapper);
            } else {
                parentWrapperList.add(itemList.get(i));
            }
        }
	}
	return parentWrapperList;
}
```
- ParentWrapper

```
boolean mIsExpanded:是否展开
long mStableId:id
Object mParentObject:parent实体
```