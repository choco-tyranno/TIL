# Android Architecture Components

+ AAC는 테스트와 유지관리가 쉬운 앱을 디자인하도록 돕는 라이브러리 모음.

  -> UI 컴포넌트 라이프사이클 관리, 데이터 지속성 처리 등.

+ [앱 아키텍처 가이드](https://developer.android.com/jetpack/guide?hl=ko&authuser=1) 참고.

+ 앱 라이프사이클 관리. 라이프사이클 인식 컴포넌트로 액티비티와 프래그먼트의 라이프사이클 관리, 구성 변경 유지, 메모리 릭을 방지, UI에 쉽게 데이터 로드.

+ LiveData를 사용해 기본 데이터베이스가 변경되면 뷰에 알리는 데이터 객체 빌드.

+ ViewModel로 앱 회전 시 제거되지 않는 UI 관련 데이터 저장.

+ Room(SQLite 개체 매핑 라이브러리)를 사용해 보일러플레이트를 피하고 SQLite 테이블 데이터를 자바 객체로 쉽게 변환. Room 은 SQLite 쿼리 컴파일 타임 확인을 제공하며 RxJava, Flowable, LiveData, Observable을 반환할 수 있음.

+ [app navigation](https://developer.android.com/guide/navigation)으로 UX를 개선.

+  DI로 보일러플레이트를 줄이고 코드를 더 쉽게 유지 관리.