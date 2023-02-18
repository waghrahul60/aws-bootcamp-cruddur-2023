FROM python:3.10-slim-buster

# Inside Container
WORKDIR /backend-flask

# Outside Container -> Inside Container
# This contail libraries want to run the app
COPY requirements.txt requirements.txt

# Inside Container
# Install the python libraries used for app
RUN pip3 install -r requirements.txt

# . mens everything in the current directory
# first period . => /backend-flask (outside)
# second period . => /backend-flask (inside)
COPY . .

# Inside container and will remain there when container is running
ENV FLASK_ENV=development

EXPOSE ${PORT}

# python3 -m flask run --host=0.0.0.0 --port=4567
CMD [ "python3", "-m" , "flask", "run", "--host=0.0.0.0", "--port=4567"]