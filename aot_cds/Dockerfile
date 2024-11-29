# 1. 빌드 단계
FROM bellsoft/liberica-runtime-container:jdk-21-cds-slim-musl AS build
WORKDIR /app

# Gradle Wrapper 복사 및 실행 권한 부여
COPY gradlew /app/
COPY gradle /app/gradle
RUN chmod +x ./gradlew

# Gradle 설정 파일 및 buildSrc 복사 후 의존성 설치
COPY build.gradle.kts settings.gradle gradle.properties /app/
COPY buildSrc /app/buildSrc
RUN ./gradlew dependencies --parallel --stacktrace

# 소스 코드 복사
COPY . /app

# 애플리케이션 빌드
RUN ./gradlew bootJar -Pspring.aot.enabled=true --no-daemon --build-cache --stacktrace


# 2. CDS 아카이브 생성 단계
FROM bellsoft/liberica-runtime-container:jre-21-cds-slim-musl AS cds
WORKDIR /app

# 빌드된 JAR 파일 복사
COPY --from=build /app/build/libs/app.jar /app/app.jar

# CDS 아카이브 생성
RUN java -XX:ArchiveClassesAtExit=./app-cds.jsa \
         -Dspring.aot.enabled=true \
         -Dspring.profiles.active=cds \
         -Dspring.context.exit=onRefresh \
         -jar app.jar


# 3. 런타임 단계
FROM bellsoft/liberica-runtime-container:jre-21-cds-slim-musl
WORKDIR /app

# 빌드된 JAR 파일 복사
COPY --from=build /app/build/libs/app.jar /app/app.jar

# CDS 아카이브 파일 복사
COPY --from=cds /app/app-cds.jsa ./

# 포트 설정 및 애플리케이션 실행
EXPOSE 8080

ENTRYPOINT ["java", "-XX:SharedArchiveFile=app-cds.jsa", "-Dspring.aot.enabled=true", "-Dspring.profiles.active=prod", "-jar", "app.jar"]
