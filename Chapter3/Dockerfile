FROM python:3-stretch

RUN pip3 install flask

COPY hello.py /tmp/hello.py

EXPOSE 5000

CMD ["python3", "/tmp/hello.py"]
