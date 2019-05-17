notepad 笔记本应用及拓展<br>
- 基本功能<br>

  1.时间戳<br>
  ![](https://github.com/wrh0619/NotePad/blob/master/images/%E6%97%B6%E9%97%B4%E6%88%B3.JPG)<br>
  相关代码：<br>
```
  <TextView
        android:id="@+id/text1_time"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:textAppearance="?android:attr/textAppearanceSmall"
        android:paddingLeft="5dip"
        android:textColor="@color/colorBlack"/>

```
  2.搜索功能<br>
  ![](https://github.com/wrh0619/NotePad/blob/master/images/%E6%90%9C%E7%B4%A2.JPG)<br>
  相关代码：<br>
  ```
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
  ```
  
- 拓展功能

  1. UI美化<br>
![](https://github.com/wrh0619/NotePad/blob/master/images/%E6%9B%B4%E6%8D%A2%E8%83%8C%E6%99%AF.JPG)<br>
![](https://github.com/wrh0619/NotePad/blob/master/images/UI%E7%BE%8E%E5%8C%96.JPG)<br>
相关代码：<br>
```
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

```
  2. 笔记排序功能<br>
  ![](https://github.com/wrh0619/NotePad/blob/master/images/%E6%8E%92%E5%BA%8F.JPG)<br>
  相关代码：<br>
  ```
   //最近打开时间排序
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
           //按创建时间排序
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

  3.字体更改<br>
![](https://github.com/wrh0619/NotePad/blob/master/images/%E5%AD%97%E4%BD%93.JPG)<br>
```
<menu xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:id="@+id/menu_save"
          android:icon="@drawable/ic_menu_save"
          android:alphabeticShortcut='s'
          android:title="@string/menu_save"
          android:showAsAction="always" />

    <item android:id="@+id/menu_revert"
          android:icon="@drawable/ic_menu_revert"
          android:title="@string/menu_revert" />

    <item android:id="@+id/menu_delete"
          android:icon="@drawable/ic_menu_delete"
          android:title="@string/menu_delete"
          android:showAsAction="always" />

    <item android:id="@+id/menu_output"
          android:title="@string/menu_output" />
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

    <item android:id="@+id/menu_color"
        android:title="@string/menu_color"
        android:icon="@drawable/ic_menu_color"
        android:showAsAction="always"/>
</menu>
```

