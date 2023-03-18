FROM python:3.9

WORKDIR /app

# Copy the current directory contents into the container at /app
COPY . /app


RUN pip install --trusted-host pypi.python.org -r requirements.txt


# Make port 80 available to the world outside this container
EXPOSE 80

# Define environment variable
ENV NAME World

CMD [ "python", "app_web.py" ]
