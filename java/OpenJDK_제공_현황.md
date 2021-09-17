# OpenJDK 제공 현황

- 레퍼런스 [https://en.wikipedia.org/wiki/OpenJDK](https://en.wikipedia.org/wiki/OpenJDK)

## OpenJDK 제공 업체 몇군데

- AdoptOpenJDK – [https://adoptopenjdk.net](https://adoptopenjdk.net/)
- Amazon–Corretto (`TCK`) – [https://aws.amazon.com/corretto](https://aws.amazon.com/corretto)
- Azul-Zulu (`TCK`) - [https://www.azul.com/downloads/zulu/](https://www.azul.com/downloads/zulu/)
- BellSoft-Liberica JDK (`TCK`) – [https://bell-sw.com/java.html](https://bell-sw.com/java.html)
- Oracle-Java SE (`TCK`) – [https://www.oracle.com/java/technologies/downloads/](https://www.oracle.com/java/technologies/downloads/)
- Oracle-OpenJDK (`TCK`) – [http://jdk.java.net](http://jdk.java.net/)
- ojdkbuild – [https://github.com/ojdkbuild/ojdkbuild](https://github.com/ojdkbuild/ojdkbuild)
- Red Hat-build of OpenJDK (`TCK`) - [https://developers.redhat.com/products/openjdk/overview](https://developers.redhat.com/products/openjdk/overview)
- SAP-SapMachine (`TCK`) – [https://sap.github.io/SapMachine](https://sap.github.io/SapMachine)
- `TCK`: Technology Compatibility Kit

G1GC의 다음 세대 GC로 오라클은 Z GC를 구현하고 있고, 다른 배급자인 레드햇은 Shenandoah GC를 이끌고 있다.


## TCK 인증을 받은 대표 OpenJDK 구현체

- Amazon Corretto 17 다운로드 링크 -> [https://docs.aws.amazon.com/corretto/latest/corretto-17-ug/downloads-list.html](https://docs.aws.amazon.com/corretto/latest/corretto-17-ug/downloads-list.html)
    - OpenJDK 17+35 에 맞춰 업데이트 된 버전이다.(OpenJDK17 의 가장 최신 버전)
    - coretto 깃헙: [https://github.com/corretto/corretto-17/blob/release-17.0.0.35.2/CHANGELOG.md](https://github.com/corretto/corretto-17/blob/release-17.0.0.35.2/CHANGELOG.md)
    - OpenJDK 깃헙: [https://github.com/openjdk/jdk17/releases/tag/jdk-17%2B35](https://github.com/openjdk/jdk17/releases/tag/jdk-17%2B35)
- Oracle OpenJDK 17 다운로드 링크 -> [https://jdk.java.net/17/](https://jdk.java.net/17/)
- Redhat OpenJDK 다운로드 링크 -> [https://developers.redhat.com/products/openjdk/download](https://developers.redhat.com/products/openjdk/download) (아직 11까지밖에 안나옴)
    - Line 에서 oracle, redhat, adopt, azul 비교 후 Redhat 으로 결정

## TCK 인증을 못받은 대표 OpenJDK 구현체

- 커뮤니티에서 유지 관리중이다.
- AdoptOpenJdk 다운로드 링크 -> [https://adoptopenjdk.net/](https://adoptopenjdk.net/) (아직 16까지밖에 안나옴)


- [Notion link](https://jennyuni.notion.site/OpenJDK-ddfff6f8244a4bdea48791cdd54dfb80x)
