# 以下简述todolist pro的"todo"部分代码实现过程

## 一.定义数据库名，版本；创建数据库
```
public class TodoDbHelper extends SQLiteOpenHelper {

    public TodoDbHelper(Context context) {
        super(context, "todo", null, 1);
    }

    @Override
    public void onCreate(SQLiteDatabase db) {
        db.execSQL(SQL_CREATE_ENTRIES);
    }

    @Override
    public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
        db.execSQL(SQL_DELETE_ENTRIES);
        onCreate(db);
    }
}
```
## 二.定义数据库表结构和 SQL 语句常量
```
public final class TodoContract {

    public static class TodoEntry implements BaseColumns {
        public static final String TABLE_NAME = "TodoEvent";
        public static final String COLUMN_NAME_CONTENT = "content";
        public static final String COLUMN_NAME_DATE = "date";
        public static final String COLUMN_NAME_STATE = "state";
        public static final String COLUMN_NAME_PRIORITY = "priority";
    }

    private TodoContract() { }

    public static final String SQL_CREATE_ENTRIES = "CREATE TABLE " +
            TodoEntry.TABLE_NAME + "(" + TodoEntry._ID +
            " INTEGER PRIMARY KEY, " + TodoEntry.COLUMN_NAME_CONTENT +
            " TEXT, " + TodoEntry.COLUMN_NAME_DATE + " TEXT, " +
            TodoEntry.COLUMN_NAME_STATE + " INTEGER, " +
            TodoEntry.COLUMN_NAME_PRIORITY + " INTEGER" + ")";

    public static final String SQL_DELETE_ENTRIES =
            "DROP TABLE IF EXIST " + TodoEntry.TABLE_NAME;
}
```
## 三.更改Note类，在Note中加入priority变量
```
private int priority;
public void setPriority(int priority) { this.priority = priority; }
public int getPriority() { return priority; }
```
## 四. 修改布局文件activity_note.xml
加入RadioGroup，包括三个RadioButton，用来选择事务的优先级，分别是emergency，important，todo
```
    <RadioGroup
        android:id="@+id/rg_priority"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content">
        <RadioButton
            android:id="@+id/rb1_urgent"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="urgent"
            android:textSize="25dp"
            android:textColor="#DF1C1C" />

        <RadioButton
            android:id="@+id/rb2_important"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="important"
            android:textSize="25dp"
            android:textColor="#FF9800"/>

        <RadioButton
            android:checked="true"
            android:id="@+id/rb3_todo"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="todo"
            android:textSize="25dp"
            android:textColor="#000"/>

    </RadioGroup>
```
## 五.完成NoteActivity.java
插入数据并判断是否返回成功
```
  private boolean saveNote2Database(String content, int priority) {
        // TODO 插入一条新数据，返回是否插入成功
        SQLiteDatabase db = dbHelper.getWritableDatabase();
        ContentValues values = new ContentValues();
        values.put(TodoContract.TodoEntry.COLUMN_NAME_CONTENT, content);
        values.put(TodoContract.TodoEntry.COLUMN_NAME_DATE, myFMT.format(new Date()));
        values.put(TodoContract.TodoEntry.COLUMN_NAME_STATE, State.TODO.intValue);
        values.put(TodoContract.TodoEntry.COLUMN_NAME_PRIORITY, priority);
        long newRowId = db.insert(TodoContract.TodoEntry.TABLE_NAME, null, values);
        if (newRowId == -1)
            return false;
        return true;
    }
```
```
private static final SimpleDateFormat myFMT =
            new SimpleDateFormat("EEE, d MMM yyyy HH:mm:ss", Locale.ENGLISH);
```
```
radioGroup = findViewById(R.id.rg_priority);
```
```
boolean succeed = saveNote2Database(content.toString().trim(), radioGroup.getCheckedRadioButtonId());
```
## 六.完成MainActivity中的数据库查询，更新，删除操作，从而实现todolist的展示，状态变化的更新和todolist中事务的删除
```
    private List<Note> loadNotesFromDatabase() {
        // TODO 从数据库中查询数据，并转换成 JavaBeans
        SQLiteDatabase db = dbHelper.getReadableDatabase();
        String[] projection = {
                TodoContract.TodoEntry._ID,
                TodoContract.TodoEntry.COLUMN_NAME_CONTENT,
                TodoContract.TodoEntry.COLUMN_NAME_DATE,
                TodoContract.TodoEntry.COLUMN_NAME_STATE,
                TodoContract.TodoEntry.COLUMN_NAME_PRIORITY
        };

        String sortOrder = TodoContract.TodoEntry.COLUMN_NAME_PRIORITY;

        Cursor cursor = db.query(TodoContract.TodoEntry.TABLE_NAME,
                projection,null, null, null,
                null, sortOrder);

        List<Note> result = new ArrayList<>();
        while(cursor.moveToNext()) {
            Note note = new Note(cursor.getLong(cursor.getColumnIndexOrThrow(TodoContract.TodoEntry._ID)));

            note.setContent(cursor.getString(cursor.getColumnIndexOrThrow(TodoContract.TodoEntry.COLUMN_NAME_CONTENT)));

            try {
                note.setDate(myFMT.parse(cursor.getString(cursor.getColumnIndexOrThrow(TodoContract.TodoEntry.COLUMN_NAME_DATE))));
            } catch (ParseException e) {
                e.printStackTrace();
            }

            note.setPriority(cursor.getInt(cursor.getColumnIndexOrThrow(TodoContract.TodoEntry.COLUMN_NAME_PRIORITY)));

            note.setState(State.from(cursor.getInt(cursor.getColumnIndexOrThrow(TodoContract.TodoEntry.COLUMN_NAME_STATE))));

            result.add(note);
        }
        return result;
    }

    private void deleteNote(Note note) {
        // TODO 删除数据
        SQLiteDatabase db = dbHelper.getWritableDatabase();
        String selection = TodoContract.TodoEntry._ID + " LIKE ?";
        String[] selectionArgs = {String.valueOf(note.id)};
        db.delete(TodoContract.TodoEntry.TABLE_NAME, selection, selectionArgs);
        notesAdapter.refresh(loadNotesFromDatabase());
    }

    private void updateNode(Note note) {
        // TODO 更新数据
        SQLiteDatabase db = dbHelper.getWritableDatabase();
        ContentValues values = new ContentValues();
        values.put(TodoContract.TodoEntry.COLUMN_NAME_STATE, note.getState().intValue);
        String selection = TodoContract.TodoEntry._ID + " LIKE ?";
        String[] selectionArgs = {String.valueOf(note.id)};
        db.update(TodoContract.TodoEntry.TABLE_NAME, values, selection, selectionArgs);
        notesAdapter.refresh(loadNotesFromDatabase());
    }
```
## 七.
