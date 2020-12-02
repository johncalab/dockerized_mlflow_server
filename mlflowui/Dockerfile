FROM python:3.8-slim-buster
RUN pip install mlflow boto3 psycopg2-binary pymysql
CMD mlflow server --host 0.0.0.0 --default-artifact-root ${DEFAULT_ARTIFACT_ROOT} --backend-store-uri ${BACKEND_STORE_URI}
