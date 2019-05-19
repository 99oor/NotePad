# NotePad应用
---
## 基本功能
- 显示笔记修改条目时间<br>
![](/screenshot/main.png)<br>

- 笔记搜索<br>
![](/screenshot/search1.png)<br>

## 扩展功能
- UI美化
- 笔记排序
- 编辑笔记字体
- 背景更换
- 导出笔记

## 功能解析
- 显示笔记修改条目时间<br>
在noteslist_item.xml布局文件中，增加一个TextView用于显示时间，并设置字体大小小于标题。<br>
```
<?xml version="1.0" encoding="utf-8"?>
<!--添加一个垂直的线性布局-->
<LinearLayout  xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/layout"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">
    <!--原标题TextView-->
    <TextView xmlns:android="http://schemas.android.com/apk/res/android"
        android:id="@android:id/text1"
        android:layout_width="match_parent"
        android:layout_height="?android:attr/listPreferredItemHeight"
        android:textAppearance="?android:attr/textAppearanceLarge"
        android:gravity="center_vertical"
        android:paddingLeft="5dip"
        android:textColor="@color/colorBlack"
        android:singleLine="true"
        />
    <!--添加显示时间的TextView-->
    <TextView
        android:id="@+id/text1_time"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:textAppearance="?android:attr/textAppearanceSmall"
        android:paddingLeft="5dip"
        android:textColor="@color/colorBlack"/>
</LinearLayout>
```
在NoteList中的PROJECTION中定义显示的时间。<br>
```
 private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
            
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE,  //显示时间
            NotePad.Notes.COLUMN_NAME_BACK_COLOR, //显示颜色
    };
```
在dataColumns，viewIDs中补充时间部分：<br>
```
String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE ,  NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE } ;
int[] viewIDs = { android.R.id.text1 , R.id.text1_time };
```
在NotePadProvider中的insert方法和NoteEditor中的updateNote方法中添加。<br>
```
Long now = Long.valueOf(System.currentTimeMillis());
Date date = new Date(now);
SimpleDateFormat format = new SimpleDateFormat("yyyy.MM.dd HH:mm:ss");
String dateTime = format.format(date);
```
运行效果：<br>
![](screenshot/example1.png)<br>


- 笔记查询

在list_options_menu.xml，添加一个item，并添加图标：<br>
```
<item
    android:id="@+id/menu_search"
    android:title="@string/menu_search"
    android:icon="@android:drawable/ic_search_category_default"
    android:showAsAction="always">
</item>
```
在NotesList中onOptionsItemSelected方法，添加搜索的case语句:<br>
```
 //添加搜素
    case R.id.menu_search:
    Intent intent = new Intent();
    intent.setClass(NotesList.this,NoteSearch.class);
    NotesList.this.startActivity(intent);
    return true;
```
菜单：<br>
![](/screenshot/example1.png)<br>

新建一个名为NoteSearch的activity。用搜索控件：SearchView。先布局搜索页面，在layout中新建布局文件note_search_list.xml：<br>
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical" android:layout_width="match_parent"
    android:layout_height="match_parent">
    <SearchView
        android:id="@+id/search_view"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:iconifiedByDefault="false"
        android:queryHint="输入搜索内容..."
        android:layout_alignParentTop="true">
    </SearchView>
    <ListView
        android:id="@android:id/list"
        android:layout_width="match_parent"
        android:layout_height="wrap_content">
    </ListView>
</LinearLayout>
```
Note Search对SearchView文本变化设置监听，并实现SearchView.OnQueryTextListener接口：<br>
```
public class NoteSearch extends ListActivity  implements SearchView.OnQueryTextListener {
    private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
            
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, // 显示时间
            NotePad.Notes.COLUMN_NAME_BACK_COLOR  //显示颜色
    };
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.note_search_list);
        Intent intent = getIntent();
        if (intent.getData() == null) {
            intent.setData(NotePad.Notes.CONTENT_URI);
        }
        SearchView searchview = (SearchView)findViewById(R.id.search_view);
        //为查询文本框注册监听器
        searchview.setOnQueryTextListener(NoteSearch.this);  
    }
    @Override
    public boolean onQueryTextSubmit(String query) {
        return false;
    }
    @Override
    public boolean onQueryTextChange(String newText) {
        String selection = NotePad.Notes.COLUMN_NAME_TITLE + " Like ? ";
        String[] selectionArgs = { "%"+newText+"%" };
        Cursor cursor = managedQuery(
                getIntent().getData(),            // Use the default content URI for the provider.
                PROJECTION,                       // Return the note ID and title for each note. and modifcation date
                selection,                        // 条件左边
                selectionArgs,                    // 条件右边
                NotePad.Notes.DEFAULT_SORT_ORDER  // Use the default sort order.
        );
        String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE ,  NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE };
        int[] viewIDs = { android.R.id.text1 , R.id.text1_time };
        MyCursorAdapter adapter = new MyCursorAdapter(
                this,
                R.layout.noteslist_item,
                cursor,
                dataColumns,
                viewIDs
        );
        setListAdapter(adapter);
        return true;
    }
    @Override
    protected void onListItemClick(ListView l, View v, int position, long id) {
        // Constructs a new URI from the incoming URI and the row ID
        Uri uri = ContentUris.withAppendedId(getIntent().getData(), id);
        // Gets the action from the incoming Intent
        String action = getIntent().getAction();
        // Handles requests for note data
        if (Intent.ACTION_PICK.equals(action) || Intent.ACTION_GET_CONTENT.equals(action)) {
            // Sets the result to return to the component that called this Activity. The
            // result contains the new URI
            setResult(RESULT_OK, new Intent().setData(uri));
        } else {
            // Sends out an Intent to start an Activity that can handle ACTION_EDIT. The
            // Intent's data is the note ID URI. The effect is to call NoteEdit.
            startActivity(new Intent(Intent.ACTION_EDIT, uri));
        }
    }
}
```
最后要在AndroidManifest.xml注册NoteSearch：<br>
```
<!--搜索activity-->
    <activity
        android:name="NoteSearch"
        android:label="@string/title_notes_search">
    </activity>
```
运行效果：<br>
搜索页面：<br>
![](/screenshot/search1.png)<br>
搜索结果：<br>
![](/screenshot/search.png)<br>


- UI美化

将NotesList从黑色换成白色，在AndroidManifest.xml中NotesList的Activity中添加：<br>
```
android:theme="@android:style/Theme.Holo.Light"
```
运行效果：<br>
![](/screenshot/main1.png)<br>


- 笔记排序

在菜单文件list_options_menu.xml中添加：<br>
```
<item
    android:id="@+id/menu_sort"
    android:title="@string/menu_sort">
    <menu>
        <item
            android:id="@+id/menu_sort1"
            android:title="@string/menu_sort1"/>
        <item
            android:id="@+id/menu_sort2"
            android:title="@string/menu_sort2"/>
        </menu>
    </item>
```
在NotesList中onOptionsItemSelected方法，添加case语句:：<br>
```
//创建时间排序
    case R.id.menu_sort1:
        cursor = managedQuery(
                getIntent().getData(),            
                PROJECTION,                      
                null,                          
                null,                          
                NotePad.Notes._ID 
                );
        adapter = new MyCursorAdapter(
                this,
                R.layout.noteslist_item,
                cursor,
                dataColumns,
                viewIDs
        );
        setListAdapter(adapter);
        return true;
 //修改时间排序
    case R.id.menu_sort2:
        cursor = managedQuery(
                getIntent().getData(),          
                PROJECTION,                      
                null,                            
                null,                       
                NotePad.Notes.DEFAULT_SORT_ORDER 
        );
        adapter = new MyCursorAdapter(
                this,
                R.layout.noteslist_item,
                cursor,
                dataColumns,
                viewIDs
        );
        setListAdapter(adapter);
        return true;
```
运行效果：<br>
菜单：<br>
![](/screenshot/list_menu.png)<br>
创建时间排序：<br>
![](/screenshot/sort.png)<br>
修改时间排序：<br>
![](/screenshot/main.png)<br>


- 编辑笔记字体

在editor_options_menu.xml，添加两个item，一个是字号菜单，一个是字体颜色菜单：<br>
```
<!--定义菜单项-->
    <item
        android:id="@+id/menu_font_size"
        android:title="@string/menu_font_size">
        <!--子菜单-->
        <menu>
            <!--定义多个菜单项-->
            <item
                android:id="@+id/font_10"
                android:title="@string/font_10"/>
            <item
                android:id="@+id/font_15"
                android:title="@string/font_15"/>
            <item
                android:id="@+id/font_20"
                android:title="@string/font_20"/>
        </menu>
    </item>

    <item
        android:id="@+id/menu_font_color"
        android:title="@string/menu_font_color" >
        <menu>
            <item
                android:id="@+id/black_font"
                android:title="@string/black_font"/>
            <item
                android:id="@+id/red_font"
                android:title="@string/red_font" />
            <item
                android:id="@+id/blue_font"
                android:title="@string/blue_font"/>
        </menu>
    </item>
```
在NotesEditor中onOptionsItemSelected方法，添加搜索的case语句:<br>
```
 //修改字体大小
        case R.id.font_10:
            txt.setTextSize(15);
            break;
        case R.id.font_15:
            txt.setTextSize(25);
            break;
        case R.id.font_20:
            txt.setTextSize(35);
            break;
        //修改字体颜色
        case R.id.black_font:
            txt.setTextColor(Color.BLACK);
            break;
        case R.id.red_font:
            txt.setTextColor(Color.RED);
            break;
        case R.id.blue_font:
            txt.setTextColor(Color.BLUE);
            break;
```
运行效果：<br>
菜单：<br>
![](/screenshot/content_menu.png)<br>
字号菜单：<br>
![](/screenshot/change_font.png)<br>
颜色菜单：<br>
![](/screenshot/change_color.png)<br>
结果：<br>
![](/screenshot/change_success.png)<br>


- 背景更换（UI美化）

主要是让NotesList和NoteSearch每条笔记都有背景色，并且能保存。<br>
先在菜单文件中添加一个更改背景的选项，editor_options_menu.xml，并添加图片：<br>
```
<item android:id="@+id/menu_color"
        android:title="@string/menu_color"
        android:icon="@drawable/ic_menu_color"
        android:showAsAction="always"/>
```
对选择的颜色界面进行布局，新建布局note_color.xml，采用ImageButton来选择颜色。在AndroidManifest.xml中将这个Acitvity主题定义为对话框样式：<br>
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="horizontal" android:layout_width="match_parent"
    android:layout_height="match_parent">
    <ImageButton
        android:id="@+id/color_white"
        android:layout_width="0dp"
        android:layout_height="50dp"
        android:layout_weight="1"
        android:background="@color/colorWhite"
        android:onClick="white"/>
    <ImageButton
        android:id="@+id/color_yellow"
        android:layout_width="0dp"
        android:layout_height="50dp"
        android:layout_weight="1"
        android:background="@color/colorYellow"
        android:onClick="yellow"/>
    <ImageButton
        android:id="@+id/color_blue"
        android:layout_width="0dp"
        android:layout_height="50dp"
        android:layout_weight="1"
        android:background="@color/colorBlue"
        android:onClick="blue"/>
    <ImageButton
        android:id="@+id/color_green"
        android:layout_width="0dp"
        android:layout_height="50dp"
        android:layout_weight="1"
        android:background="@color/colorGreen"
        android:onClick="green"/>
    <ImageButton
        android:id="@+id/color_red"
        android:layout_width="0dp"
        android:layout_height="50dp"
        android:layout_weight="1"
        android:background="@color/colorRed"
        android:onClick="red"/>
</LinearLayout>
```
在NotePadProvide的onCreate方法中添加颜色的字段：<br>
```
    NotePad.Notes.COLUMN_NAME_BACK_COLOR + " INTEGER" //颜色
```
把颜色定义为INTEGER的主要原因是，方便颜色的对应与使用：<br>
```
public static final int DEFAULT_COLOR = 0; //白
public static final int PINK_COLOR = 1; //粉
public static final int BLUE_COLOR = 2; //蓝
public static final int GREEN_COLOR = 3; //绿
public static final int RED_COLOR = 4; //红
public static final int PURPLE_COLOR = 5; //紫
```
insert中：<br>
```
 // 新建笔记，背景默认为白色
if (values.containsKey(NotePad.Notes.COLUMN_NAME_BACK_COLOR) == false) {
        values.put(NotePad.Notes.COLUMN_NAME_BACK_COLOR, NotePad.Notes.DEFAULT_COLOR);
        }
```
将颜色填充到ListView，用SimpleCursorAdapter中的bindView方法来实现。定义一个CursorAdapter继承SimpleCursorAdapter，完成数据和颜色的填充：<br>
```
public class MyCursorAdapter extends SimpleCursorAdapter {


    public MyCursorAdapter(Context context, int layout, Cursor c,
                           String[] from, int[] to) {
        super(context, layout, c, from, to);
    }


    @Override
    public void bindView(View view, Context context, Cursor cursor){
        super.bindView(view, context, cursor);
        //从数据库中读取的cursor中获取笔记列表对应的颜色数据，并设置笔记颜色
        int x = cursor.getInt(cursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_BACK_COLOR));
        /**
         * 白 255 255 255
         * 粉 250 209 225
         * 蓝 165 202 237
         * 绿 161 214 174
         * 红 255 119 146
         * 紫 157 175 252
         */
        switch (x){
            case NotePad.Notes.DEFAULT_COLOR:
                view.setBackgroundColor(Color.rgb(255, 255, 255));
                break;
            case NotePad.Notes.PINK_COLOR:
                view.setBackgroundColor(Color.rgb(250,209,225));
                break;
            case NotePad.Notes.BLUE_COLOR:
                view.setBackgroundColor(Color.rgb(165, 202, 237));
                break;
            case NotePad.Notes.GREEN_COLOR:
                view.setBackgroundColor(Color.rgb(161, 214, 174));
                break;
            case NotePad.Notes.RED_COLOR:
                view.setBackgroundColor(Color.rgb(255,119,146));
                break;
            case NotePad.Notes.PURPLE_COLOR:
                view.setBackgroundColor(Color.rgb(157,175,252));
                break;
            default:
                view.setBackgroundColor(Color.rgb(255, 255, 255));
                break;
        }
    }
}
```
将NotesList中用的SimpleCursorAdapter改使用MyCursorAdapter：<br>
```
 //修改为可以填充颜色的自定义的adapter，自定义的代码在MyCursorAdapter.java中
adapter = new MyCursorAdapter(
        this,
        R.layout.noteslist_item,
        cursor,
        dataColumns,
        viewIDs
    );
```

在NoteEditor中的PROJECTION添加颜色项：<br>
```
  private static final String[] PROJECTION =
        new String[] {
            NotePad.Notes._ID,
            NotePad.Notes.COLUMN_NAME_TITLE,
            NotePad.Notes.COLUMN_NAME_NOTE,
            NotePad.Notes.COLUMN_NAME_BACK_COLOR
    };
```
在NoteEditor类中有onResume()方法中添加。<br>
```
    //读取颜色数据做准备
    int x = mCursor.getInt(mCursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_BACK_COLOR));
    /**
     * 白 255 255 255
     * 粉 250 209 225
     * 蓝 165 202 237
     * 绿 161 214 174
     * 红 255 119 146
     * 紫 157 175 252
     */
    switch (x){
        case NotePad.Notes.DEFAULT_COLOR:
            mText.setBackgroundColor(Color.rgb(255, 255, 255));
            break;
        case NotePad.Notes.PINK_COLOR:
            mText.setBackgroundColor(Color.rgb(250,209,225));
            break;
        case NotePad.Notes.BLUE_COLOR:
            mText.setBackgroundColor(Color.rgb(165, 202, 237));
            break;
        case NotePad.Notes.GREEN_COLOR:
            mText.setBackgroundColor(Color.rgb(161, 214, 174));
            break;
        case NotePad.Notes.RED_COLOR:
            mText.setBackgroundColor(Color.rgb(255,119,146));
            break;
        case NotePad.Notes.PURPLE_COLOR:
            mText.setBackgroundColor(Color.rgb(157,175,252));
            break;
        default:
            mText.setBackgroundColor(Color.rgb(255, 255, 255));
            break;
    }
```
在NoteEditor中找到onOptionsItemSelected()方法，在菜单的switch中添加case：<br>
```
//换背景颜色选项
    case R.id.menu_color:
        changeColor();
        break;
```
在NoteEditor中添加函数changeColor()：<br>
```
//跳转改变颜色的activity，将uri信息传到新的activity
    private final void changeColor() {
        Intent intent = new Intent(null,mUri);
        intent.setClass(NoteEditor.this,NoteColor.class);
        NoteEditor.this.startActivity(intent);
    }
```
NoteColor.java
```
public class NoteColor extends Activity {
    private Cursor mCursor;
    private Uri mUri;
    private int color;
    private static final int COLUMN_INDEX_TITLE = 1;
    private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_BACK_COLOR,
    };
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.note_color);
        //从NoteEditor传入的uri
        mUri = getIntent().getData();
        mCursor = managedQuery(
                mUri,        // The URI for the note that is to be retrieved.
                PROJECTION,  // The columns to retrieve
                null,        // No selection criteria are used, so no where columns are needed.
                null,        // No where columns are used, so no where values are needed.
                null         // No sort order is needed.
        );
    }
    @Override
    protected void onResume(){
    //执行顺序在onCreate之后
        if (mCursor != null) {
            mCursor.moveToFirst();
            color = mCursor.getInt(COLUMN_INDEX_TITLE);
        }
        super.onResume();
    }
    @Override
    protected void onPause() {
    //执行顺序在finish()之后，将选择的颜色存入数据库
        super.onPause();
        ContentValues values = new ContentValues();
        values.put(NotePad.Notes.COLUMN_NAME_BACK_COLOR, color);
        getContentResolver().update(mUri, values, null, null);
    }
    public void white(View view){
        color = NotePad.Notes.DEFAULT_COLOR;
        finish();
    }
    public void yellow(View view){
        color = NotePad.Notes.YELLOW_COLOR;
        finish();
    }
    public void blue(View view){
        color = NotePad.Notes.BLUE_COLOR;
        finish();
    }
    public void green(View view){
        color = NotePad.Notes.GREEN_COLOR;
        finish();
    }
    public void red(View view){
        color = NotePad.Notes.RED_COLOR;
        finish();
    }

}
```
AndroidManifest.xml中定义颜色选择对话框.<br>
```
<!--换背景色-->
<activity android:name="NoteColor"
    android:theme="@android:style/Theme.Holo.Light.Dialog"
    android:label="ChangeColor"
    android:windowSoftInputMode="stateVisible"/>
```
运行效果：<br>
颜色选择对话框：<br>
![](/screenshot/color.png)<br>
编辑页面：<br>
![](/screenshot/background1.png)<br>
主页面：<br>
![](/screenshot/background2.png)<br>


- 导出笔记

在菜单文件中添加一个导出笔记的选项，editor_options_menu.xml：<br>
```
<item android:id="@+id/menu_output"
    android:title="@string/menu_output" />
```
新建布局output_text.xml，用来显示一个导出对话框。：<br>

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:orientation="vertical"
    android:paddingLeft="6dip"
    android:paddingRight="6dip"
    android:paddingBottom="3dip">
    <EditText android:id="@+id/output_name"
        android:maxLines="1"
        android:layout_marginTop="2dp"
        android:layout_marginBottom="15dp"
        android:layout_width="wrap_content"
        android:ems="25"
        android:layout_height="wrap_content"
        android:autoText="true"
        android:capitalize="sentences"
        android:scrollHorizontally="true" />
    <Button android:id="@+id/output_ok"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="right"
        android:text="@string/output_ok"
        android:onClick="OutputOk" />
</LinearLayout>
```
在NoteEditor中找到onOptionsItemSelected()方法，在菜单的switch中添加case：<br>
```
//导出笔记选项
   case R.id.menu_output:
        outputNote();
        break;
```
在NoteEditor中添加函数outputNote()：<br>
```
//跳转导出笔记的activity，将uri信息传到新的activity
    private final void outputNote() {
        Intent intent = new Intent(null,mUri);
        intent.setClass(NoteEditor.this,OutputText.class);
        NoteEditor.this.startActivity(intent);
    }
```

```
public class OutputText extends Activity {
   //要使用的数据库中笔记的信息
    private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
            NotePad.Notes.COLUMN_NAME_NOTE, // 2
            NotePad.Notes.COLUMN_NAME_CREATE_DATE, // 3
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, // 4
    };
    //读取出的值放入这些变量
    private String TITLE;
    private String NOTE;
    private String CREATE_DATE;
    private String MODIFICATION_DATE;
    //读取该笔记信息
    private Cursor mCursor;
    //导出文件的名字
    private EditText mName;
    //NoteEditor传入的uri，用于从数据库查出该笔记
    private Uri mUri;
    //关于返回与保存按钮的一个特殊标记，返回的话不执行导出，点击按钮才导出
    private boolean flag = false;
    private static final int COLUMN_INDEX_TITLE = 1;
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.output_text);
        mUri = getIntent().getData();
        mCursor = managedQuery(
                mUri,        // The URI for the note that is to be retrieved.
                PROJECTION,  // The columns to retrieve
                null,        // No selection criteria are used, so no where columns are needed.
                null,        // No where columns are used, so no where values are needed.
                null         // No sort order is needed.
        );
        mName = (EditText) findViewById(R.id.output_name);
    }
    @Override
    protected void onResume(){
        super.onResume();
        if (mCursor != null) {
            // The Cursor was just retrieved, so its index is set to one record *before* the first
            // record retrieved. This moves it to the first record.
            mCursor.moveToFirst();
            //编辑框默认的文件名为标题，可自行更改
            mName.setText(mCursor.getString(COLUMN_INDEX_TITLE));
        }
    }
    @Override
    protected void onPause() {
        super.onPause();
        if (mCursor != null) {
        //从mCursor读取对应值
            TITLE = mCursor.getString(mCursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_TITLE));
            NOTE = mCursor.getString(mCursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_NOTE));
            CREATE_DATE = mCursor.getString(mCursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_CREATE_DATE));
            MODIFICATION_DATE = mCursor.getString(mCursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE));
            //flag在点击导出按钮时会设置为true，执行写文件
            if (flag == true) {
                write();
            }
            flag = false;
        }
    }
    public void OutputOk(View v){
        flag = true;
        finish();
    }
    private void write()
    {
        try
        {
            // 如果手机插入了SD卡，而且应用程序具有访问SD的权限
            if (Environment.getExternalStorageState().equals(
                    Environment.MEDIA_MOUNTED)) {
                // 获取SD卡的目录
                File sdCardDir = Environment.getExternalStorageDirectory();
                //创建文件目录
                File targetFile = new File(sdCardDir.getCanonicalPath() + "/" + mName.getText() + ".txt");
                //写文件
                PrintWriter ps = new PrintWriter(new OutputStreamWriter(new FileOutputStream(targetFile), "UTF-8"));
                ps.println(TITLE);
                ps.println(NOTE);
                ps.println("创建时间：" + CREATE_DATE);
                ps.println("最后一次修改时间：" + MODIFICATION_DATE);
                ps.close();
                Toast.makeText(this, "保存成功,保存位置：" + sdCardDir.getCanonicalPath() + "/" + mName.getText() + ".txt", Toast.LENGTH_LONG).show();
            }
        }
        catch (Exception e)
        {
            e.printStackTrace();
        }
    }
}
```
在AndroidManifest.xml中将这个Acitvity主题定义为对话框样式，并且加入权限：<br>
```
<!--添加导出activity-->
        <activity android:name="OutputText"
            android:label="@string/output_name"
            android:theme="@android:style/Theme.Holo.Dialog"
            android:windowSoftInputMode="stateVisible">
        </activity>
```
```
 <!-- 在SD卡中创建与删除文件权限 -->
    <uses-permission android:name="android.permission.MOUNT_UNMOUNT_FILESYSTEMS"/>
    <!-- 向SD卡写入数据权限 -->
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
```
运行效果：<br>
点击隐藏的菜单，选择导出笔记：<br>
 ![](/screenshot/content_menu.png)<br>
可修改文件名，点击确认导出：<br>
![](/screenshot/output.png)<br>
保存成功，弹出提示：<br>
![](/screenshot/output_success.png)<br>
查看文件：<br>
![](/screenshot/phone_file.png)<br>
点击刚导出的文件，用阅读器打开查看内容：<br>
![](/screenshot/file.png)<br>

