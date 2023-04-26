# ECR image push sample

## 사전조건
1. Docker 설치
2. AWS cli 설치 (version 2.x 이상)




## Docker image build

#### Dockerfile이 존재하는 디렉토리로 이동
```
$ cd [Dockerfile이 위치한 디렉토리명]
```
#### 이미지 생성
```
$ docker build -t [image name] .
```

#### 이미지 생성본 확인
```
$ docker images
```


## SSO named profile 구성

SSO란?
Single Sign-On(SSO)은 1회 사용자 인증으로 다수의 애플리케이션 및 웹사이트에 대한 사용자 로그인을 허용하는 인증 솔루션

(ref: https://aws.amazon.com/ko/what-is/sso/)

#### SSO 세션 연결

```
$ aws configure sso

SSO session name (Recommended): [세션 이름]         //선택사항
SSO start URL [None]: [랜딩존 주소]                 //https://d-9b67193bea.awsapps.com/start/ 와 같이 입력
SSO region [None]: [이용하는 region]                //ap-northeast-2 
SSO registration scopes [sso:account:access]:       //엔터치고 넘어가도 무방
```

```
Attempting to automatically open the SSO authorization page in your default browser.
If the browser does not open or you wish to use a different device to authorize this request, open the following URL:

[https://device.sso.ap-northeast-2.amazonaws.com/]        //해당URL을 브라우저에 입력

Then enter the code:

[접속 코드]                  //브라우저 페이지에 해당 코드를 입력
```


#### profile 구성
```
CLI default client Region [None]: [기본 region]                      //ap-northeast-2 
CLI default output format [None]: [기본 출력 형식]                   //json
CLI profile name [Administrator-123456789123]: [기본 프로파일]       //my-dev-profile
```

#### 완료메세지 확인
이 프로파일을 사용하려면 아래 결과 메세지와 같이 --profile을 사용하여 프로파일을 지정한다
```
To use this profile, specify the profile name using --profile, as shown:

aws s3 ls --profile my-dev-profile
```
#### profile 확인
아래 경로의 config 파일에서 방금 생성한 named profile 정보를 확인할 수 있다.
```
$ vim ~/.aws/config
```


## SSO 로그인

#### 생성한 프로파일을 사용
```
$ aws sso login --profile [생성한 프로파일]        //my-dev-profile
```

```
Attempting to automatically open the SSO authorization page in your default browser.
If the browser does not open or you wish to use a different device to authorize this request, open the following URL:

https://device.sso.ap-northeast-2.amazonaws.com/        //해당URL을 브라우저에 입력

Then enter the code:

[접속 코드]                  //브라우저 페이지에 해당 코드를 입력
```

#### SSO 로그인 완료
코드 입력후 브라우저 창에서 Allow 클릭 후 콘솔에서 아래 메세지 확인
```
Successfully logged into Start URL: https://d-9b67193bea.awsapps.com/start/
```

#### SSO profile을 활용하여 명령 호출 테스트
```
$ aws sts get-caller-identity --profile [생성한 프로파일]
{
    "UserId": "userId",
    "Account": "accountNumber",
    "Arn": "arn"
}

```

## ECR 로그인

```
$ aws ecr get-login-password --profile [생성한 프로파일] --region [사용자 region] | docker login --username AWS --password-stdin [계정번호(12자리)].dkr.ecr.[사용자 region].amazonaws.com

// aws ecr get-login-password --profile my-dev-profile --region ap-northeast-2 | docker login --username AWS --password-stdin 123456789123.dkr.ecr.ap-northeast-2.amazonaws.com
```

## ECR repository 생성
#### repository 생성
옵션은 환경에 맞춰서 사용
```
aws ecr create-repository \
    --repository-name [생성할 repository 이름] \
    --image-scanning-configuration scanOnPush=false \
    --region [사용자 region] \
    --profile [생성한 프로파일]
```

#### repository 정보 출력 확인
```
{
    "repository": {
        "repositoryArn": "arn:aws:ecr:ap-northeast-2:123456789123:repository/hello-repository",
        "registryId": "123456789123",
        "repositoryName": "hello-repository",
        "repositoryUri": "123456789123.dkr.ecr.ap-northeast-2.amazonaws.com/hello-repository",
        "createdAt": "2023-03-09T08:52:38+00:00",
        "imageTagMutability": "MUTABLE",
        "imageScanningConfiguration": {
            "scanOnPush": false
        },
        "encryptionConfiguration": {
            "encryptionType": "AES256"
        }
    }
}
```

## ECR repository Imange push

#### Image push를 위한 tag변경
```
docker tag [위에서 생성한 이미지 이름] [repository URL]:[이미지 버전 태그]
docker tag [위에서 생성한 이미지 이름] [repository URL]:latest

//
docker tag hello-world 123456789123.dkr.ecr.ap-northeast-2.amazonaws.com/hello-repository:0.0.1
docker tag hello-world 123456789123.dkr.ecr.ap-northeast-2.amazonaws.com/hello-repository:latest
```

#### Image push
```
docker push [repository URL]:[이미지 버전 태그]
docker push [repository URL]:[latest]

//
docker push 123456789123.dkr.ecr.ap-northeast-2.amazonaws.com/hello-repository:0.0.1
docker push 123456789123.dkr.ecr.ap-northeast-2.amazonaws.com/hello-repository:latest
```
