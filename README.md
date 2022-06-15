Hello everyone, welcome to another tutorial, I'm gonna share with you how to push docker image to docker hub using Github actions.
before we start i assume that you have basic understanding of [docker](https://www.docker.com/) and [github](https://github.com/) .



## What is github actions:

GitHub Actions enables the user to create custom Software Development Life Cycle (SDLC) workflows in their GitHub repositories. It gives a privilege to the repository owner to write individual actions and then combine 
them to create a custom workflow of their choice for their project. GitHub Actions also provides the facility to build end-to-end Continuous Integration (CI) and Continuous Deployment (CD) capabilities directly in the repository.


---

Let us take an example of how GitHub Action can be implemented in a simple repository. 

#### Step 1 

1. create a new repository
**add image here**

2. Create Dockerfile

```
nano Dockerfile
```

paste this code in Dockerfile
```
FROM python:3.6

ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1


ADD . /code
WORKDIR /code

RUN pip install -r requirements.txt

CMD ["python", "app.py"]
```

3. Then Create app.py and paste this code below

```
from flask import Flask

app = Flask(__name__)

@app.route('/')
def home():
    return 'Home Page'

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=8070, debug=False)
```

4. now let's create requirements.txt file to install required python packages

```
nano requirements.txt
```
And also  paste this code below.

```
flask=2.0.2
```

To make sure everything is work perfectly, run the following command.

```bash
$ docker build -t  push-docker-image:latest .
```

then

```
docker run -d  -p 8070:8070 push-docker-image:latest
```

Now check on the web browser 

**browser image**


#### Step 2
Now i'll create github actions config file


1. Create .github/workflows folder.

```
mkdir -p .github/workflows
```

2. and inside workflows folder create a .yml file, (eg push-docker-image.yaml)

```
touch .github/workflows/push-docker-image.yaml
```
paste this code into your .yml file

```
name: This a workflow title 

on: [push] # When pushing to any branch then run this action


# Env variable
env:
  DOCKER_USER: ${{secrets.DOCKER_USER}}
  DOCKER_PASSWORD: ${{secrets.DOCKER_PASSWORD}}
  REPO_NAME: ${{secrets.REPO_NAME}}

jobs:

  push-image-to-docker-hub:  # job name

    runs-on: ubuntu-latest  # runner name : (ubuntu latest version) 

    steps:
    - uses: actions/checkout@v2 # first action : checkout source code
    - name: docker login
      run: | # log into docker hub account
        docker login -u $DOCKER_USER -p $DOCKER_PASSWORD  
    - name: Get current date # get the date of the build
      id: date
      run: echo "::set-output name=date::$(date +'%Y-%m-%d--%M-%S')"

    - name: Build the Docker image # push The image to the docker hub
      run: docker build . --file Dockerfile --tag $DOCKER_USER/$REPO_NAME:${{ steps.date.outputs.date }}
 
    - name: Docker Push
      run: docker push $DOCKER_USER/$REPO_NAME:${{ steps.date.outputs.date }}

```


After pushing the code to github, go to the **Actions** sectio, you will find the workflow you created, you will receive an error because thoes variables DOCKER_USER and DOCKER_PASSWORD and REPO_NAME are not set.

In order to fix that, go to settings --> secrets and create tree secrets (DOCKER_USER & DOCKER_PASSWORD, REPO_NAME), re-push an update and go back to the actions section you will now see a green circle witch means the workflow was completed successfully.

Now go to docker hub you will find an images with a tag similaire to `YYYY-M-D--m-s`. congratulation we have successfully push an image to docker hub.


Resources

* https://docs.github.com/en/actions
