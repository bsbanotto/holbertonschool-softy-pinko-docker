# Download base image ubuntu 22.04
FROM ubuntu:22.04

# Update APT Repository
RUN apt-get update
RUN apt-get upgrade -y

# Install Python and pip3
RUN apt-get install python3 -y
RUN apt-get install python3-pip -y

# Install Flask
RUN pip3 install flask
RUN pip3 install flask-cors

# Create Working
WORKDIR /app

COPY ./api.py /app/api.py

# Echo 'Hello, World!'
CMD ["python3", "api.py"]
