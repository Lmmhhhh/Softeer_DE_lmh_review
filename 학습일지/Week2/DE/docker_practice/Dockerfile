# 1. 베이스 이미지
FROM python:3.11-slim

# 2. 작업 디렉터리
WORKDIR /app

# 3. 의존성 복사 및 설치
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# 4. 애플리케이션 코드 복사
COPY app.py .

# 5. 문서화용 포트 표시
EXPOSE 5000

# 6. 컨테이너 시작 시 실행할 명령
CMD [ "python", "app.py" ]
