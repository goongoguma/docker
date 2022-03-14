- Creating and Running a Container from an Image
  ```
  docker run  <image name>
  ```

- overriding default command
  <command>는 container에서 실행될 명령어
  ```
  docker run <image name> <command>
  // ex. docker run busybox ls
  ```

- List all running containers
  ```
  docker ps
  ```
  만들었던 모든 container 확인
  ```
  docker ps -a
  ```

- Container lifecylcle
  - docker run은 docker create와 docker start를 합쳐놓은 명령어
  - create a container
    ```
    docker create <image name>
    ```
  - start a container
    ```
    docker start <container id>
    // container의 결과를 터미널에 표시하고 싶으면 docker start -a <container id>
    ``` 

- Restarting stopped container
  ```
  docker start -a <container id>
  ```

- Removing stopped container
  - 사용하지않고있는 모든 container를 삭제
  ```
  docker system prune
  ```

- Retrieving log outputs
  ```
  docker logs <container id>
  ```

- Stopping container
  - stop a container (대부분)
  - 점진적으로 container stop 
  - stop 명령어를 실행하면 container를 멈추기 위해 다양한 내부 작업들이 진행됨
  ```
  docker stop <container id>
  ```
  - kill a container
  - 급진적으로 container stop
  - 바로 중지함
  ```
  docker kill <container id>
  ```
  - docker stop이 실행된 후, 10초동안 container가 멈추지 않으면 자동적으로 kill container 명령어를 실행한다

- Executing commands in running containers
  ```
  // -it 명령어를 사용해서 container안에 명령어를 전달
  docker exec -it <container id> <command>

  // redis 서버가 돌아가는 container안에서 redis-cli 실행
  ex. docker exec -it f5cf02 redis-cli
  ```
 
- Getting a command prompt in a container
  - container에서 명령어를 실행할 때마다 docker exec 명령어를 실행하는것은 너무 번거로움
  - sh 명령어를 사용해서 unix로 들어감
  ```
  ex. docker exec -it <contaienr id> sh
  ```
  - sh(shell)는 터미널, 커맨드 프롬트라는 뜻

- Docker Teardown
  ```  
  // FROM,RUN,CMD instruction telling Docker Server what to do
  FROM alpine

  RUN apk add --update redis

  CMD ["redis-server"]

  ```
  - FROM: docker image we want to use as a base
  - RUN: run execute command
  - CMD: what should be executed when our image is used to start up brand new container

- What is Base Image?
  - Writing a dockerfile == Being given a computer with no OS and being told to install Chrome
  - How do you install Chrome on a computer with no operation system?<br />
    step1. Install an operating system<br />
    step2. Start up your default browser<br />
    step3. Navigate to chrome.google.com<br />
    step4. Downlaod installer<br />
    step5. Open file/folder explorer<br />
    step6. Execute chrome_installer.exe<br />
    step7. Execute chrome.exe<br />
  - Docker image 생성 단계로 비유하자면<br />
    step1은 specify a base image (alpine을 OS로 생각하면됨)<br />
    step2부터 step6단계는 run commands to install additional programs<br />
    step7은 command to run on startup<br />
  - Why did we use alpine as a base image?<br />
    -> Why do you use Windows, MacOS, or Ubuntu? -> They come with a preinstalled set of programs that are useful to you!<br />
    - We are trying to install, run redis and alpine base image has a set of program inside of it that are very useful for installing and running redis

- The Build Process in Detail
  - docker build . -> Dockerfile을 사용해 image 생성

- Taggin an Image
  - docker run <image id>
  - 하지만 docker run을 실행할 때마다 id를 복붙하는 작업은 번거롭다
  - 그렇기에 image를 태그해서 docker run hello-world와 같이 이미지 실행을 간소화 할 수 있음
  - 아래 명령어와 같이하면됨
  ```
  docker build -t goongamja/redis-test:latest .
  // goongamja -> 내 docker id
  // redis-test -> 내가 만든 repo 혹은 프로젝트의 이름
  // latest -> 버전 
  // . -> 빌드하기 위한 파일이나 폴더의 경로
  ```
  - 명령어 실행 결과물 
  ```
  => => writing image sha256:c7fe01986ccc9bb76c6b7dac506724aff2c68e77c833c598fa5c63b4e1b8900e
  => => naming to docker.io/goongamja/redis-test:latest
  ```
  - 이후로 image를 실행하기 위해서 아래의 명령어를 입력하면 된다.
  ```
  docker run goongamja/redis-test
  ```
  
- Base Image Issues
  - 프로젝트에서 아래 코드로 image를 빌드하면 에러가 발생함
  ```
   # Specify a base image
    FROM alpine
    WORKDIR /usr/app

    # Install some dependencies
    RUN npm install

    # Default command
    CMD [ "npm", "start" ]
  ```
  - 이유는 alpine 프로그램에 node가 없기때문임
  - 그렇기 때문에 docker repository에서 만들어진 node:alpine을 사용하면된다.
  ```
  # Specify a base image
  FROM node:alpine
  WORKDIR /usr/app

  # Install some dependencies
  RUN npm install

  # Default command
  CMD [ "npm", "start" ]
  ```
  - 하지만 이렇게 하면 NOENT: no such file or directory, open '/usr/app/package.json' 에러가 발생하는데 이유는 container안에 package.json파일이 존재하지 않기 때문이다. container는 하드드라이브의 일부분에서 작동하는것일뿐 하드의 다른 부분과 연결되어 있지 못하기 때문에 루트 경로에 있는 만들어진 package.json과 연결되지않는다.  

- Copying Build Files
  - 위의 문제를 해결하기 위해 COPY 명령어를 사용할것
  - COPY instruction is used to move files and folders from our local file system on your machine to the file system inside of temporary container that is created during the build process.
  ```
  COPY ./ ./
  // 첫번째 ./ -> Path to folder to copy from on your machine (relative to build context)
  // 두번째 ./ -> Place to copy stuff to inside the container
  ```
  - COPY 명령어를 입력하고 이미지를 빌드 및 실행을 하고 docker run후에 Listening on port 8080을 확인하였으나 해당 포트에 접속하면 사이트에 연결할 수 없음이 뜬다. 왜 그럴까

- Container Port Mapping
  - 포트 8080은 container안에서만 실행되고 있기 때문에 로컬에서의 8080 접속이 안되는거였음
  - 로컬 컴퓨터에서 container안의 포트에 접속하기 위해서는 port mapping 작업이 필요함
  - 이 작업을 위해 container를 실행하는 docker run 명령어를 수정해야 함
  ```
  docker run -p 5000:8080 <container id or name>
  // 첫번째 8080 -> Route incoming requests to this port on local host to...
  // 두번째 8080 -> this port inside the container
  ```

- Specifying a Working Directory
  - docker run -it goongamja/simpleweb sh를 이용해 container안으로 들어가 ls를 사용하면 해당 container안에 다양한 파일과 폴더들이 있는것을 알 수 있다.
  - 이런점은 나중에 파일이나 폴더를 추가하였을때 같은 이름으로 생성할시 기존의 파일과 폴더이름 충돌이 발생할 수 있는 확률이 있기때문에 바람직하지 않다. 그렇기에 WORKDIR 명령어를 추가한다.
  ```
  WORKDIR /usr/app
  // any following command will be executed relative to this path in the container
  ```
- Unnecessary Rebuilds
  - 만약 index.js파일을 수정하고 docker를 다시 빌드하게 된다면...
  - index.js의 내용만 수정했을뿐이지만 npm이 재설치된다.

- Minimizing Cache Busting and Rebuilds
  - 이 문제를 해결하기 위해 다음과 같이 Dockerfile을 수정한다.
  ```
  // package.json 파일이 변경될때만 npm install을 실행한다
  COPY ./package.json ./
  RUN npm install
  COPY ./ ./
  ```
  - 추가적으로 팁은 도커환경에서 앱을 실행 시킬때는 로컬환경에 있는 node_modules 파일은 지워도 된다. Dockerfile에서 npm install 시 자동적으로 node_modules가 생기기 때문에 로컬에 있는 node_modules는 삭제(빌드 COPY시 시간이 소요됨)

- Docker Compose
  - Separate CLI that gets installed along with Docker
  - Used to start up multiple Docker containers at the same time and auomatically connect them together
  - Automates some of the long-winded arguments we were passing to 'docker run'

- Docker Compose Files
  - docker compose 파일을 이용해서 아래와 같은 명령어들을
  ```
  docker build -t goongamja/visits:latest (node:alpine 실행)
  docker run -p 8080:8080 goongamja/visits
  ```
  docker-compose.yml 파일하나에 작성해서 docker-cli에게 넘길것
  ```
  // docker-compose.yml
  만들고자 하는 container:
    redis-server
      Make it using the 'redis' image
    node-app
      Make it using the Dockerfile in the current directory
      Map port 8081 to 8081
  ```
  - 실제예시
  ```
  version: '3'
  services: 
    redis-server: 
      image: 'redis'
    node-app: 
    # . -> look in the current directory for a Dockerfile and use that to build the image
      build: .
      ports:
        - "4001:8081"
  ```

- Docker compose command
  - compose file에 있는 모든 container, image, service의 이미지를 만들기 위해서 docker-compose up 명령어를 사용한다.
  - Dockerfile의 이미지를 다시 빌드하고 실행하기 위해서는 docker-compose up --build 명령어를 사용한다.
  - docker-compose up으로 visits 프로젝트가 실행된다. 

- Stopping Docker Compose Container
  - Launch in background
  ```
  docker-compose up -d
  ```
  - Stop Containers
  ```
  docker-compose down
  ```

- Automatic Container Restarts
  - docker compose가 container를 다시 시작하는 기준
    - Restart Policies
      - "no": Never attempt to restart this . contaienr if it stops or crashes
      - always: If this container stops *for any reason* always attempt to restart it
      - on-failure: Only restart if the container stops with an error code
      - unless-stopped: Always restart unless we (the developers) forcibly stop it
    ```
    // 예시
    version: '3'
    services: 
      redis-server: 
        image: 'redis'
      node-app: 
        restart: always
        build: .
        ports:
          - "4001:8081"
    ```
- Container Status with Docker Compose 
  - docker-compose ps 명령어로 경로에 있는 compose파일의 container 상태를 확인할 수 있다.

- Creating the Dev DockerFile
  - 개발 버전을 위해 Dockerfile.dev를 설정하고
  ```
  FROM node:16-alpine

  WORKDIR '/app'

  COPY package.json .
  RUN npm install

  COPY . .

  CMD ["npm", "run", "start"]
  ```
  docker build . 명령어를 실행했으나 Dockerfile을 찾을 수 없다는 에러가 발생한다.<br />
  이럴때는 docker build -f Dockerfile.dev . 로 실행해야 한다. (f는 파일 설정)

- Docker Volumes
  - Dockerfile.dev를 빌드후에 docker run -p 3000:3000 id를 하게되면 프로젝트가 시작된다.
  - 하지만 로컬 파일에서 파일이 변하더라도 해당 변화가 localhost:3000에 적용되지 않는다. 이 문제를 해결하기 위해 docker volumes를 사용할것.
  - docker volumes를 사용해서 docker container가 로컬 폴더에 있는 경로들을 참조하게 만들것.
  - 명령어가 조금 복잡하다.
  ```
  // pwd -> present working directory
  docker run -p 3000:3000 -v /app/node_modules -v$(pwd):/app <image_id>

  // /app/node_modules -> placeholder (:가 없다면 container 밖에 참조할 내용이 없는것)
  // -v$(pwd):/app -> take everything inside working directory and map it up to the app folder inside of the container
  ```
  - 아래의 명령어를 실행하면 프로젝트가 실행되고 수정사항이 실시간 반영되는것을 확인할 수 있다. 
  ```
  docker run -p 3000:3000 -v /app/node_modules -v $(pwd):/app 17cf375c5bfcf60ce0baea05fee2bce85eb9b055d43104fe15c59068c8600276
  ```
- Shorthand with Docker Compose
  - 하지만 명령어가 너무 길다. 
  - docker-compose.yml 파일을 작성하자
  ```
  version: '3'
  services:
    web: 
      build: .
      ports: 
        - "3000:3000"
      volumes:
        - /app/node_modules
        # . -> outside folder
        # /app -> inside container
        - .:/app
  ```
  - docker-compose up을 해도 작동하지 않는다. 왜냐하면 현재 경로에 Dockerfile이 아닌 Dockerfile.dev 파일이 있기 때문이다.
  - 그렇기 때문에 Dockerfile 대신 Dockerfile.dev 파일을 컴포즈할 방법을 찾아야 한다. 

- Overriding Dockerfile Selection
  - context와 dockerfile을 build 아래에 추가한다.
  ```
  version: '3'
  services:
    web: 
      build:
        # 현재 경로에서 Dockerfile.dev를 찾아 빌드
        context: .
        dockerfile: Dockerfile.dev
      ports: 
        - "3000:3000"
      volumes:
        - /app/node_modules
        # . -> outside folder
        # /app -> inside container
        - .:/app

  ```
