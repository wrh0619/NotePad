# notepad 笔记本应用及拓展

## 一、项目主要文件目录说明



1. 类文件说明

![](http://ww1.sinaimg.cn/large/8cc1690dgy1g2sjsx9shrj208n08x0sy.jpg)

- notepad类

  这个类为契约类，里面定义相关数据库字段属性，配置信息等等

- NotePadProvider类 提供数据的访问

- MyCursorAdapter 适配器类

- NoteColor 类 颜色的显示

- NoteList 主页面列表的显示

- TitleEditor 标题的编辑

- NoteEditor 笔记内容编写的activity



## 二、基本功能及其拓展

- 基本功能

  1. 增加时间戳功能

  2. 笔记内容的搜索功能

- 拓展功能

  1. 笔记排序功能

  2. UI美化

  3. 字体颜色设置及字体大小

     

- 详细说明

  1. 时间戳功能

     首先在notelist_item.xml 中设置用户显示时间的TextView，添加如下代码：

     ```
         <!--添加 显示时间 的TextView-->
         <TextView
             android:id="@+id/text1_time"
             android:layout_width="match_parent"
             android:layout_height="wrap_content"
             android:textAppearance="?android:attr/textAppearanceSmall"
             android:paddingLeft="5dip"
             android:textColor="@color/colorBlack"/>
     ```

     因为数据库当中已经存在有时间的信息，所以我们并不需要对数据库进行修改。

     之后在NoteList文件中定义要显示的属性，添加修改时间，如下代码中：

     ```
         /**
          * The columns needed by the cursor adapter
          */
         private static final String[] PROJECTION = new String[] {
                 NotePad.Notes._ID, // 0
                 NotePad.Notes.COLUMN_NAME_TITLE, // 1
                 NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, // 2
                 NotePad.Notes.COLUMN_NAME_BACK_COLOR,
         };
     ```

     添加上面代码中的

     ```
     NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, // 2
     ```

     在dataColumns，viewIDs中补充时间部分：

     ```
        private String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE ,  NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE } ;
         private int[] viewIDs = { android.R.id.text1 , R.id.text1_time };
     ```

     在**NotePadProvider中的insert方法**和**NoteEditor中的** 对日期的显示格式进行修改

     ```
     // 将milliseconds转化为一定的时间格式
     Long now = Long.valueOf(System.currentTimeMillis());
     Date date = new Date(now);
     SimpleDateFormat format = new SimpleDateFormat("yyyy.MM.dd HH:mm:ss");
     String dateTime = format.format(date);
     ```

     至此，时间戳的功能已经完成。如下所示：

     ![](http://ww1.sinaimg.cn/large/8cc1690dgy1g31s9dxx65j209q0g8abh.jpg)

  2.  笔记内容搜索功能

     首先在list_options_menu.xml 中添加笔记查询的菜单

     ```
         <!--菜单中增加一个搜索项-->
         <item
             android:id="@+id/menu_search"
             android:title="@string/menu_search"
             android:icon="@android:drawable/ic_search_category_default"
             android:showAsAction="always">
         </item>
     ```

     在NoteList的onOptionsItemSelected方法中添加：

     ```
                 //添加搜索
                 case R.id.menu_search:
                     Intent intent = new Intent();
                     intent.setClass(NotesList.this,NoteSearch.class);
                     NotesList.this.startActivity(intent);
                     return true;
     ```

     新建一个NoteSearch的Activity文件：

     ```
     package com.example.android.notepad;
     
     import android.app.ListActivity;
     import android.content.ContentUris;
     import android.content.Intent;
     import android.database.Cursor;
     import android.net.Uri;
     import android.os.Bundle;
     import android.view.View;
     import android.widget.ListView;
     import android.widget.SearchView;
     import android.widget.SimpleCursorAdapter;
     
     
     
     
     public class NoteSearch extends ListActivity  implements SearchView.OnQueryTextListener {
     
         private static final String[] PROJECTION = new String[] {
                 NotePad.Notes._ID, // 0
                 NotePad.Notes.COLUMN_NAME_TITLE, // 1
                 //扩展 显示时间 颜色
                 NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, // 2
                 NotePad.Notes.COLUMN_NAME_BACK_COLOR
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
             searchview.setOnQueryTextListener(NoteSearch.this);  //为查询文本框注册监听器
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
                     selection,
                     selectionArgs,
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

     实现了SearchView.OnQueryTextListener ，可以动态显示搜索的内容。

     效果如下图所示：

     

![](http://ww1.sinaimg.cn/large/8cc1690dgy1g31ununcamj209q0g8jst.jpg)

![](http://ww1.sinaimg.cn/large/8cc1690dgy1g31up07qt4j20a60ha3zj.jpg)

![](http://ww1.sinaimg.cn/large/8cc1690dgy1g31upua929j209q0g8jsk.jpg)

​		由于使用了

```
	String[] selectionArgs = { "%"+newText+"%" };
```

​		可以实现模糊查询。

 3. 笔记排序功能

    在list_options_menu.xml 中添加

    ```
    <item
            android:id="@+id/menu_sort"
            android:title="@string/menu_sort"
            android:icon="@android:drawable/ic_menu_sort_by_size"
            android:showAsAction="always" >
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

    之后在NoteList当中的switch语句中添加：

    ```
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
    
    ```

    该功能比较简单，主要修改cursor中的排序方式就可以完成！

    效果如下所示：

    ![](http://ww1.sinaimg.cn/large/8cc1690dgy1g31v38c2b6j20a60hatad.jpg)

![](http://ww1.sinaimg.cn/large/8cc1690dgy1g31v40zxqej20a60hamyu.jpg)

![](http://ww1.sinaimg.cn/large/8cc1690dgy1g31v4hthsnj209q0g8q4d.jpg)

4. UI美化功能

   更改app的主题

   在manifest文件中更改APP的主题：

   ```
   android:theme="@android:style/Theme.Holo.Light"
   ```

   添加数据表中的颜色字段：

   ```
    @Override
       public void onCreate(SQLiteDatabase db) {
           db.execSQL("CREATE TABLE " + NotePad.Notes.TABLE_NAME + "   ("
           + NotePad.Notes._ID + " INTEGER PRIMARY KEY,"
           + NotePad.Notes.COLUMN_NAME_TITLE + " TEXT,"
           + NotePad.Notes.COLUMN_NAME_NOTE + " TEXT,"
           + NotePad.Notes.COLUMN_NAME_CREATE_DATE + " INTEGER,"
           + NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE + " INTEGER,"
           + NotePad.Notes.COLUMN_NAME_BACK_COLOR + " INTEGER" //颜色
           + ");");
          }
   ```

   然后在契约类当中添加契约信息：

   ```
   public static final int DEFAULT_COLOR = 0; //白
   public static final int YELLOW_COLOR = 1; //黄
   public static final int BLUE_COLOR = 2; //蓝
   public static final int GREEN_COLOR = 3; //绿
   public static final int RED_COLOR = 4; //红
   ```

   在notePrivater 文件中static执行相应的修改：

   ```
   sNotesProjectionMap.put(
           NotePad.Notes.COLUMN_NAME_BACK_COLOR,
           NotePad.Notes.COLUMN_NAME_BACK_COLOR);
   ```

   在insert当中：

   ```
   if (values.containsKey(NotePad.Notes.COLUMN_NAME_BACK_COLOR) == false) {
           values.put(NotePad.Notes.COLUMN_NAME_BACK_COLOR, NotePad.Notes.DEFAULT_COLOR);
           }
   ```

   新建一个adapter继承simpleAdapter ， 可以进行颜色的填充：

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
            * 黄 247 216 133
            * 蓝 165 202 237
            * 绿 161 214 174
            * 红 244 149 133
            */
           switch (x){
               case NotePad.Notes.DEFAULT_COLOR:
                   view.setBackgroundColor(Color.rgb(255, 255, 255));
                   break;
               case NotePad.Notes.YELLOW_COLOR:
                   view.setBackgroundColor(Color.rgb(247, 216, 133));
                   break;
               case NotePad.Notes.BLUE_COLOR:
                   view.setBackgroundColor(Color.rgb(165, 202, 237));
                   break;
               case NotePad.Notes.GREEN_COLOR:
                   view.setBackgroundColor(Color.rgb(161, 214, 174));
                   break;
               case NotePad.Notes.RED_COLOR:
                   view.setBackgroundColor(Color.rgb(244, 149, 133));
                   break;
               default:
                   view.setBackgroundColor(Color.rgb(255, 255, 255));
                   break;
           }
       }
   }
   ```

   将之前的simpleadapter全部替换，然后在noteList当中添加颜色的显示：

   ```
   private static final String[] PROJECTION = new String[] {
               NotePad.Notes._ID, // 0
               NotePad.Notes.COLUMN_NAME_TITLE, // 1
               //扩展 显示时间 颜色
               NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, // 2
               NotePad.Notes.COLUMN_NAME_BACK_COLOR,
       };
   ```

   之后做编辑笔记时的背景色更换

   在NoteEditor 当中添加如下更改如下的代码：

   ```
     private static final String[] PROJECTION =
           new String[] {
               NotePad.Notes._ID,
               NotePad.Notes.COLUMN_NAME_TITLE,
               NotePad.Notes.COLUMN_NAME_NOTE,
               NotePad.Notes.COLUMN_NAME_BACK_COLOR
       };
   ```

   在onResume()中添加如下代码：

   ```
   //读取颜色数据做准备
   int x = mCursor.getInt(mCursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_BACK_COLOR));
   /**
    * 白 255 255 255
    * 黄 247 216 133
    * 蓝 165 202 237
    * 绿 161 214 174
    * 红 244 149 133
    */
   switch (x){
       case NotePad.Notes.DEFAULT_COLOR:
           mText.setBackgroundColor(Color.rgb(255, 255, 255));
           break;
       case NotePad.Notes.YELLOW_COLOR:
           mText.setBackgroundColor(Color.rgb(247, 216, 133));
           break;
       case NotePad.Notes.BLUE_COLOR:
           mText.setBackgroundColor(Color.rgb(165, 202, 237));
           break;
       case NotePad.Notes.GREEN_COLOR:
           mText.setBackgroundColor(Color.rgb(161, 214, 174));
           break;
       case NotePad.Notes.RED_COLOR:
           mText.setBackgroundColor(Color.rgb(244, 149, 133));
           break;
       default:
           mText.setBackgroundColor(Color.rgb(255, 255, 255));
           break;
   }
   ```

在editor_options_menu.xml当中添加一个菜单背景的菜单。



```
<item android:id="@+id/menu_color"
        android:title="@string/menu_color"
        android:icon="@drawable/ic_menu_color"
        android:showAsAction="always"/>
```

在NoteEditor中找到onOptionsItemSelected()方法，在菜单的switch中添加：

```
//换背景颜色选项
    case R.id.menu_color:
        changeColor();
        break;
```

在NoteEditor中添加函数changeColor()：

```
//跳转改变颜色的activity，将uri信息传到新的activity
    private final void changeColor() {
        Intent intent = new Intent(null,mUri);
        intent.setClass(NoteEditor.this,NoteColor.class);
        NoteEditor.this.startActivity(intent);
    }
```

创建一个新的activity：

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

对应的xml布局文件如下所示：

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

之后将AndroidMenefest.xml 定义如下该activity的声明：

```
<activity android:name="NoteColor"
    android:theme="@android:style/Theme.Holo.Light.Dialog"
    android:label="ChangeColor"
    android:windowSoftInputMode="stateVisible"/>
```

实现效果如下所示：

![](http://ww1.sinaimg.cn/large/8cc1690dgy1g320luv1wzj20a60hagms.jpg)

![](http://ww1.sinaimg.cn/large/8cc1690dgy1g320mlr9xpj20a60hawfu.jpg)

![](http://ww1.sinaimg.cn/large/8cc1690dgy1g320nn4tezj20a60ha0tt.jpg)

![](http://ww1.sinaimg.cn/large/8cc1690dgy1g320o4m963j20a60hadhi.jpg)

5. 设置字体的颜色和大小

首先在editor_options_menu.xml 文件中定义字体大小和颜色的菜单目录。

```
<item
    android:id="@+id/item1"
    android:title="字体大小">
    <menu>
        <item
            android:id="@+id/small"
            android:title="小">
        </item>
        <item
            android:id="@+id/middle"
            android:title="中">
        </item>
        <item
            android:id="@+id/big"
            android:title="大">
        </item>

    </menu>
</item>
<item android:id="@+id/item3"
    android:title="字体颜色">
    <menu>
        <item
            android:title="红"
            android:id="@+id/red">

        </item>
        <item android:title="黑"
            android:id="@+id/black">

        </item>
    </menu>
</item>
```

之后在NoteEditor中onOptionsItemSelected方法添加如下代码：

```
EditText editText = findViewById(R.id.note);
 case R.id.red:
                editText.setTextColor(getResources().getColor(R.color.red));
                break;
            case R.id.black:
                editText.setTextColor(getResources().getColor(R.color.black));
                break;
            case R.id.big:
                editText.setTextSize(20);
                break;
            case R.id.small:
                editText.setTextSize(10);
                break;
            case R.id.middle:
                editText.setTextSize(16);
                break;
```

效果如下所示：

![](http://ww1.sinaimg.cn/large/8cc1690dgy1g320zb4k94j20a60hadhf.jpg)

![](http://ww1.sinaimg.cn/large/8cc1690dgy1g3210d9q87j20a60hamyk.jpg)

![](http://ww1.sinaimg.cn/large/8cc1690dgy1g3211218tvj20a60ha75j.jpg)

![](http://ww1.sinaimg.cn/large/8cc1690dgy1g3211ut6d4j20a60hadh8.jpg)
