<h1>RoomDatabase.Callback</h1>

> Add Callback at databaseBuilder.

`````java
    public static MyDataBase getDatabase(final Context context) {
        if (INSTANCE == null) {
            synchronized (MyDataBase.class) {
                if (INSTANCE == null) {
                    INSTANCE = Room.databaseBuilder(context.getApplicationContext()
                            , MyDataBase.class, "my_database")
                        	//Add callback :
                            .addCallback(sRoomDatabaseCallback).build();
                }
            }
        }
        return INSTANCE;
    }
`````



> Define callback :

`````java
private static final RoomDatabase.Callback sRoomDatabaseCallback = new RoomDatabase.Callback() {
        @Override
        public void onCreate(@NonNull SupportSQLiteDatabase db) {
            super.onCreate(db);
            //Can be prepopulate line here.
        }

        @Override
        public void onOpen(@NonNull SupportSQLiteDatabase db) {
            super.onOpen(db);
        }
    };
`````

> #onCreate called when database is created just once.

> #onOpen called when database is opened.



<H3>Error : RoomDatabase.Callback doesn't called. </h3>

> cause : In this e.g., If MyDataBase instance aren't used, the database won't be created.

> solve : force use instance. e.g. DAO#findItem.