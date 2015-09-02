###SharedPreferences分析

> 使用

```
SharedPreferences sharedPreferencesContext;
SharedPreferences sharedPreferences;

sharedPreferencesContext = getSharedPreferences("test",MODE_PRIVATE);
sharedPreferences = getPreferences(MODE_PRIVATE);

SharedPreferences.Editor editor = sharedPreferencesContext.edit();
editor.putBoolean("saved",true);
Set<String> set = new HashSet<>();
set.add("aaaaa");
set.add("bbbbbbb");
editor.putStringSet("content",set);
editor.commit();

sharedPreferences.Editor editorActivity = sharedPreferences.edit();
editorActivity/putString("name","haha");
editorActivity.commit();

```
> 分析

```
|--实现类:SharedPreferencesImpl

|--Context
	|--getSharedPreferences(String name,int mode)
	
|--Activity
	|--getPreferences(int mode):以activity的名字为文件名
```