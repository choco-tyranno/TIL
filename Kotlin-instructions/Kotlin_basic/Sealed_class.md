# Sealed class

+ Abstract class로 자식 클래스의 종류를 제한하는 특성
+ 컴파일러가 sealed 클래스의 자식클래스 종류를 알수 있음
  ->when문에서 else 구현 제거 및 분기 필요를 파악가능.
+ Sealed class가 속한 직속 패키지 내에서만 상속이 가능.