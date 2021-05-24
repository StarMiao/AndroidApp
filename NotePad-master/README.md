# Notepad大实验作业：

## 功能一：添加时间戳功能

### 在布局文件中增加时间戳

1.找到布局文件noteslist_item.xml，源码如下：

```
   <RelativeLayout android:layout_height="match_parent"
    android:layout_width="match_parent"
    xmlns:android="http://schemas.android.com/apk/res/android">
    <TextView xmlns:android="http://schemas.android.com/apk/res/android"
        android:id="@android:id/text1"
        android:layout_width="match_parent"
        android:layout_height="?android:attr/listPreferredItemHeight"
        android:textAppearance="?android:attr/textAppearanceLarge"
        android:gravity="center_vertical"
        android:paddingLeft="5dip"
        android:singleLine="true"
    />
</RelativeLayout>
```
源码中默认带有一个显示标题的TextView，为了让其显示时间戳，我为其新增一个TextView用于显示时间戳，改进后代码如下：

```
<RelativeLayout android:layout_height="match_parent"
    android:layout_width="match_parent"
    xmlns:android="http://schemas.android.com/apk/res/android">
    <TextView xmlns:android="http://schemas.android.com/apk/res/android"
        android:id="@android:id/text1"
        android:layout_width="match_parent"
        android:layout_height="?android:attr/listPreferredItemHeight"
        android:textAppearance="?android:attr/textAppearanceLarge"
        android:gravity="center_vertical"
        android:paddingLeft="5dip"
        android:singleLine="true"
    />
    <TextView
        android:id="@+id/text2"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:paddingLeft="5dip"
        android:singleLine="true"
        android:gravity="center_vertical"
    />
</RelativeLayout>
```

### 将系统默认的时间显示格式更改为我们自定义时间格式，增加可读性

insert():

```
public Uri insert(Uri uri, ContentValues initialValues) {
        if (sUriMatcher.match(uri) != NOTES) {
            throw new IllegalArgumentException("Unknown URI " + uri);
        }
        ContentValues values;
        if (initialValues != null) {
            values = new ContentValues(initialValues);

        } else {
            values = new ContentValues();
        }
        Long now = Long.valueOf(System.currentTimeMillis());
        if (values.containsKey(NotePad.Notes.COLUMN_NAME_CREATE_DATE) == false) {
            values.put(NotePad.Notes.COLUMN_NAME_CREATE_DATE, now);
        }
        if (values.containsKey(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE) == false) {
            values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, now);
        }
        if (values.containsKey(NotePad.Notes.COLUMN_NAME_TITLE) == false) {
            Resources r = Resources.getSystem();
            values.put(NotePad.Notes.COLUMN_NAME_TITLE, r.getString(android.R.string.untitled));
        }
        if (values.containsKey(NotePad.Notes.COLUMN_NAME_NOTE) == false) {
            values.put(NotePad.Notes.COLUMN_NAME_NOTE, "");
        }
        SQLiteDatabase db = mOpenHelper.getWritableDatabase();
        long rowId = db.insert(
            NotePad.Notes.TABLE_NAME,
            NotePad.Notes.COLUMN_NAME_NOTE,                                         
            values
        );

        if (rowId > 0) {

            Uri noteUri = ContentUris.withAppendedId(NotePad.Notes.CONTENT_ID_URI_BASE, rowId);
            getContext().getContentResolver().notifyChange(noteUri, null);
            return noteUri;
        }
        throw new SQLException("Failed to insert row into " + uri);
    }
```

2.以上代码中对12-18行进行修改，将系统默认的时间now进行格式化，改进后的代码如下：

		//系统默认显示的时间使用毫秒数进行表示，很难阅读
		Long now = Long.valueOf(System.currentTimeMillis());
        Date date = new Date(now);
		//将时间格式化为常见的年月日表示形式
        SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yy-MM-dd HH:mm:ss");
        simpleDateFormat.setTimeZone(TimeZone.getTimeZone("GMT+08:00"));
        String dateFormat = simpleDateFormat.format(date);
        if (values.containsKey(NotePad.Notes.COLUMN_NAME_CREATE_DATE) == false) {
            values.put(NotePad.Notes.COLUMN_NAME_CREATE_DATE, dateFormat);
        }
        if (values.containsKey(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE) == false) {
            values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, dateFormat);
        }

updateNote()中:

```
private final void updateNote(String text, String title) {
        ContentValues values = new ContentValues();
        values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, System.currentTimeMillis());
        if (mState == STATE_INSERT) {
            if (title == null) {
                int length = text.length();
                title = text.substring(0, Math.min(30, length));
                if (length > 30) {
                    int lastSpace = title.lastIndexOf(' ');
                    if (lastSpace > 0) {
                        title = title.substring(0, lastSpace);
                    }
                }
            }
            values.put(NotePad.Notes.COLUMN_NAME_TITLE, title);
        } else if (title != null) {
            values.put(NotePad.Notes.COLUMN_NAME_TITLE, title);
        }
        values.put(NotePad.Notes.COLUMN_NAME_NOTE, text);
        getContentResolver().update(
                mUri,
                values, 
                null,
                null
            );
    }
 ```
 
3. 第三行代码添加默认的毫秒数时间格式，我们对其进行时间的格式化，改进后代码如下:
```
    private final void updateNote(String text, String title) {
        ContentValues values = new ContentValues();
        long now = System.currentTimeMillis();
        Date date = new Date(now);
        SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yy-MM-dd HH:mm:ss");
        simpleDateFormat.setTimeZone(TimeZone.getTimeZone("GMT+08:00"));
        String dateFormat = simpleDateFormat.format(date);
        values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, dateFormat);
        if (mState == STATE_INSERT) {

            if (title == null) {
                int length = text.length();
                title = text.substring(0, Math.min(30, length));
                if (length > 30) {
                    int lastSpace = title.lastIndexOf(' ');
                    if (lastSpace > 0) {
                        title = title.substring(0, lastSpace);
                    }
                }
            }
            values.put(NotePad.Notes.COLUMN_NAME_TITLE, title);
        } else if (title != null) {
            values.put(NotePad.Notes.COLUMN_NAME_TITLE, title);
        }
        values.put(NotePad.Notes.COLUMN_NAME_NOTE, text);
        getContentResolver().update(
                mUri,    
                values,
                null, 
                null    
            );
    }
 ```
4. 将获取到的时间进行赋值操作
观察源码我们可以发现，在SQLite中已经定义过时间变量了我们就没有必要进行修改，代码如下：
```
       public void onCreate(SQLiteDatabase db) {
           db.execSQL("CREATE TABLE " + NotePad.Notes.TABLE_NAME + " ("
                   + NotePad.Notes._ID + " INTEGER PRIMARY KEY,"
                   + NotePad.Notes.COLUMN_NAME_TITLE + " TEXT,"
                   + NotePad.Notes.COLUMN_NAME_NOTE + " TEXT,"
                   + NotePad.Notes.COLUMN_NAME_CREATE_DATE + " INTEGER,"
                   + NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE + " INTEGER"
                   + ");");
       }
```
我们进行时间变量的赋值操作：
```
if (values.containsKey(NotePad.Notes.COLUMN_NAME_CREATE_DATE) == false) {
            values.put(NotePad.Notes.COLUMN_NAME_CREATE_DATE, dateFormat);
        }
        if (values.containsKey(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE) == false) {
            values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, dateFormat);
        }
```
5. 接下来我们找到创建Note时的源码：
```
public class NotesList extends ListActivity {
    private static final String TAG = "NotesList";
    private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
    };
    private static final int COLUMN_INDEX_TITLE = 1;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setDefaultKeyMode(DEFAULT_KEYS_SHORTCUT);
        Intent intent = getIntent();
        if (intent.getData() == null) {
            intent.setData(NotePad.Notes.CONTENT_URI);
        }
        getListView().setOnCreateContextMenuListener(this);
        Cursor cursor = managedQuery(
            getIntent().getData(),
            PROJECTION,
            null,
            null,
            NotePad.Notes.DEFAULT_SORT_ORDER
        );
        String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE } ;
        int[] viewIDs = { android.R.id.text1 };
        SimpleCursorAdapter adapter
            = new SimpleCursorAdapter(
                      this,
                      R.layout.noteslist_item,
                      cursor,
                      dataColumns,
                      viewIDs
              );
        setListAdapter(adapter);
    }
```

6. 查看源码可以发现，他只是显示id以及titile，并没有时间，我们加上时间的显示，cursor就会将新增的时间一并检索，而后装载相应数据列，我们一并将其修改并显示时间戳。

```
private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE,//日期
    };
    private static final int COLUMN_INDEX_TITLE = 1;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setDefaultKeyMode(DEFAULT_KEYS_SHORTCUT);
        Intent intent = getIntent();
        if (intent.getData() == null) {
            intent.setData(NotePad.Notes.CONTENT_URI);
        }
        getListView().setOnCreateContextMenuListener(this);
        Cursor cursor = managedQuery(
            getIntent().getData(),
            PROJECTION,
            null,
            null,
            NotePad.Notes.DEFAULT_SORT_ORDER
        );
        String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE, NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE } ;
        int[] viewIDs = { android.R.id.text1, R.id.text2 }
        SimpleCursorAdapter adapter
            = new SimpleCursorAdapter(
                      this,
                      R.layout.noteslist_item,
                      cursor,
                      dataColumns,
                      viewIDs
              );
        setListAdapter(adapter);
    }

String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE, NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE };
int[] viewIDs = { android.R.id.text1, R.id.text2 };
```
运行结果
![image](https://user-images.githubusercontent.com/57957504/119346106-5d52f480-bccc-11eb-885f-91dccf7be36c.png)

### 功能二:添加查询功能
1. 添加图标
我们在顶部的菜单栏添加一个查询的图标，找到list_option_menu.xml，添加代码如下：
```
    <item
        android:id="@+id/menu_search"
        android:icon="@android:drawable/ic_search_category_default"
        android:showAsAction="always"
        android:title="search">
    </item>
```
添加选择事件
找到函数onOptionsItemSelected，添加选择事件：

```
case R.id.menu_search:
                Intent intent = new Intent();
                intent.setClass(this, NoteSearch.class);
                this.startActivity(intent);
                return true;
```

2. 新建一个用于查找功能实现以及布局的Activity
新建Activity：NoteSearch以及它的布局文件note_search.xml
编辑布局文件，代码如下：

```
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">
    <SearchView
        android:id="@+id/search_view"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:iconifiedByDefault="false"
    </SearchView>
    <ListView
        android:id="@+id/list_view"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        >
    </ListView>
</LinearLayout>
```
3. 在NoteSeach中查找功能的实现，代码如下:

```
public boolean onQueryTextChange(String newText) {    
    String selection1 = NotePad.Notes.COLUMN_NAME_TITLE+" like ? or "+NotePad.Notes.COLUMN_NAME_NOTE+" like ?";
    String[] selection2 = {"%"+string+"%","%"+string+"%"};
    Cursor cursor=sqLiteDatabase.query(
        NotePad.Notes.TABLE_NAME,
        PROJECTION,
        selection1, 
        selection2,
        null,
        null,
        NotePad.Notes.DEFAULT_SORT_ORDER
    );
    listview.setAdapter(adapter);
        return true;
}
```

4. 显示以及获取代码的实现：

```
ListView listview;
SQLiteDatabase sqLiteDatabase;
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    requestWindowFeature(Window.FEATURE_NO_TITLE);   								
    super.setContentView(R.layout.note_search);
    Intent intent = getIntent();
    if (intent.getData() == null) {
        intent.setData(NotePad.Notes.CONTENT_URI);
    }
    listview= (ListView) findViewById(R.id.list_view);//获取listview
    sqLiteDatabase=new NotePadProvider.DatabaseHelper(this).getReadableDatabase();
    SearchView search= (SearchView) findViewById(R.id.search_view);//获取搜索视图
    search.setOnQueryTextListener(NoteSearch.this);  
}
```

运行结果
![image](https://user-images.githubusercontent.com/57957504/119346081-5330f600-bccc-11eb-99fe-6c3665d7d271.png)
