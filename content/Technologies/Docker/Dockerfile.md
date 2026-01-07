
| Command                             | Description                     |
| ----------------------------------- | ------------------------------- |
| `docker build -t <New-Image-Tag> .` | build image based on dockerfile |




## syntax

| Command                             | Description                                                                                      |
| ----------------------------------- | ------------------------------------------------------------------------------------------------ |
| FROM python:latest                  | Tells docker which base image will be used.                                                      |
| WORKDIR /app                        | tells docker to change working directory, if dir not exists it will be created                   |
| COPY requirements.txt .             | Copy content from host dir to container, `copy <host/File> <container path>`                     |
| RUN pip install -r requirments.txt  |                                                                                                  |
| EXPOSE 5000                         |                                                                                                  |
| CMD python hello.py                 | This is the entry command for the container                                                      |
| `ADD <url> <Docker/path>`           | Download from web into container.                                                                |
| `ENV SQL_user=asdf password="asdf"` | Set env variables, this will be global accors all intermediate containers while building.        |
| USER slom                           | Change User to slom instead of staying in root context.                                          |
| `ENTRYPOINT ["/bin/bash", "-c"]`    | specify the base command that will be executed in runtime, its arguments will be found in `CMD`. |
| `CMD ["python3"]`                   | this specify the args of `ENTRYPOINT`, you can use CMD only or ENTRPYPOINT only.                 |
| ARG SQL_SA=sa                       | This work as variable inside dockerfile we can reuse it.                                         |

---

> [!Attention] 
> There is commands that change in layers like `COPY` which make changes in file system and every change makes new layer , but things like `CMD` works in runtime of the image "conatiner" which no need to create new layer for it, we just make metadata file contain those runtime commands.
