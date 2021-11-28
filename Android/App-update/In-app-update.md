# In app update



## Work flow 

### Flexible update :

<img src="https://t1.daumcdn.net/cfile/tistory/9929E3425D0C64C420">



### Immediate update :

<img src="https://t1.daumcdn.net/cfile/tistory/999562425D0C64E51D">



Update status : 

`````java
public @interface UpdateAvailability {
    int UNKNOWN = 0;
    int UPDATE_NOT_AVAILABLE = 1;
    int UPDATE_AVAILABLE = 2;
    int DEVELOPER_TRIGGERED_UPDATE_IN_PROGRESS = 3;
}
`````

`````java
public @interface InstallStatus {
    int UNKNOWN = 0;
    int REQUIRES_UI_INTENT = 10;
    int PENDING = 1;
    int DOWNLOADING = 2;
    int DOWNLOADED = 11;
    int INSTALLING = 3;
    int INSTALLED = 4;
    int FAILED = 5;
    int CANCELED = 6;
}
`````

