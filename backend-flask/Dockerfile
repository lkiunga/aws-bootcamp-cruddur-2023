FROM python:3.10-slim-buster

#Inside Container
#Make a new folder inside container
WORKDIR /backend-flask

#Outside Container -> Unside Container

COPY requirements.txt requirements.txt
#Install the python Library used for the app
RUN pip install -r requirements.txt
#outside Container -> Inside Container
#. means everything in the current directory
COPY . .

ENV FLASK_ENV=development

EXPOSE ${PORT}
#CMD (COMMAND)
#Used to run flask
#python3 -m  flask run --host=0.0.0.0 --port=4567
CMD [ "python3", "-m", "flask", "run", "--host=0.0.0.0", "--port=4567" ]