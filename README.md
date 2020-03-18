# 들어가며 

이 워크샵은 Amazon Personalize Samples([영문 원본](https://github.com/aws-samples/amazon-personalize-samples/tree/master/getting_started)을 한글화하였으며, 
사전 배경 지식이 필요한 고객들을 위해 SageMaker Factorization Machine로 추천 시스템을 구축하는 법을 신규로 추가하였습니다. 
총 워크샵 소요 시간은 SageMaker 핸즈온까지 수행 시 총 3시간이며, SageMaker 핸즈온을 건너 뛰고 2번부터 진행 시에는 약 2시간입니다.<br>
추천 시스템에 대한 배경 지식과 MovieLens 데이터셋에 대한 기초적인 분석이 필요하면 1번을 반드시 진행해 주세요.

# AWS 추천 시스템 워크샵

1. [Amazon SageMaker Factorization Machine(FM)으로 추천 시스템 구축하기](0.Recommendation-System-FM-KNN.ipynb)
2. [Amazon Personalize 데이타 가져오기. 약 15분 소요](1.Validating_and_Importing_User_Item_Interaction_Data.ipynb)
3. [Amazon Personalize 솔류션 생성하기, 약 40분 소요](2.Creating_and_Evaluating_Solutions.ipynb)
4. [Amazon Personalize 캠페인 생성하기, 약 10분 소요](3.Deploying_Campaigns.ipynb)
5. [Amazon Personalize 캠페인 및 Interaction 확인, 약 1분 소요](4.View_Campaign_And_Interactions.ipynb)
6. [리소스 삭제, 약 10분 소요](5.Cleanup.ipynb)

아래 가이드는 Personalize 핸즈온 실습을 위해서 필요한 환경 구성을 합니다.
주요하게 세이지 메이커 노트북 인스턴스 생성, 디폴트 Git 코드 복사, S3 버킷 생성, 역할(Role) 생성 및 권한 생성 등을 합니다.
(**만약 세이지 메이커 인스턴스가 생성이 되어 있고, 환경에 친숙하시다면 역할(Role)에 아래 4개의 Policy 없으면 추가 해주세요.<br>
그리고 이 과정은 스킵 하셔도 됩니다.** AmazonSageMakerFullAccess, AmazonS3FullAccess, AmazonPersonalizeFullAccess, IAMFullAccess)<br> 


# Getting Started


아래의 원문 영어에 대한 한글 번역은 요약만을 한 것이기에, 번역이 이해가 어려우시면 원문 영어를 보시기 바랍니다.

The tutorial below will walk you through building an environment to create a custom dataset, model, and recommendation campaign with Amazon Personalize. If you have any issues with any of the content below please open an issue here in the repository.

## Prerequisites

다음과 같은 아래의 2개의 필수 요건이 필요 합니다.<br>
Only applies if you are deploying with the CloudFormation template, otherwise consult the IAM permissions needed for your specific task.

1. AWS Account
2. User with administrator access to the AWS Account


## Building Your Environment

아래 작업은 CloudFormation Template을 구성하는 작업 입니다.<br>
* 인터넷 브라우저를 하나 더 여시고, AWS Account로 로긴하세요.
* 이후에 브라우저에 안에서 새로운 탭을 하나 생성 후에, 아래 링크를 클릭하여 CloudFormation을 통해 관련 작업을 수행 하시면 됩니다.<br>
The first step is to deploy a CloudFormation template that will perform much of the initial setup for you. In another browser window login to your AWS account. Once you have done that open the link below in a new tab to start the process of deploying the items you need via CloudFormation.

[![Launch Stack](https://s3.amazonaws.com/cloudformation-examples/cloudformation-launch-stack.png)](https://console.aws.amazon.com/cloudformation/home#/stacks/new?stackName=PersonalizeDemo&templateURL=https://gonsoomoon-priviate-share.s3.ap-northeast-2.amazonaws.com/DaekeunPersonalizeDemo.yaml)

아래 스크린 샸의 가이드데로 하시면 됩니다.<br>
Follow along with the screenshots if you have any questions about these steps.

### Cloud Formation Wizard

"Next" 버튼을 눌러서 진행 하십시오.<br>
Start by clicking `Next` at the bottom like shown:

![StackWizard](static/imgs/img1.png)

고유한 S3 버킷 이름을 지정해야 합니다. 간단하게 영어로 본인의 이름을 **영어소문자** 로 더해주세요.
(예: personalizeddemo-gonsoomoon)
이후에 'Next'버튼을 클릭 해주세요 <br>
In the next page you need to provide a unique S3 bucket name for your file storage, it is recommended to simply add your first name and last name to the end of the default option as shown below, after that update click `Next` again.

![StackWizard2](static/imgs/img3.png)

스크롤을 맨 밑까지 하신 후에 'Next' 클릭 해주세요 <br>
This page is a bit longer so scroll to the bottom to click `Next`.

![StackWizard3](static/imgs/img4.png)

맨 밑까지 스크롤 하신 후에 체크 박스 클릭(새로운 IAM 리소스 생성 관련) 해주세요.
이후에 'Create Stack'클릭 해주세요.<br>
Again scroll to the bottom, check the box to enable the template to create new IAM resources and then click `Create Stack`.

![StackWizard4](static/imgs/img5.png)

2-3분 후에 CloudFormation은 세이지 메이커 노트북 인스턴스, S3 버킷, 역할(Role) 등을 생성합니다.<br>
For a few minutes CloudFormation will be creating the resources described above on your behalf it will look like this while it is provisioning:

![StackWizard5](static/imgs/img6.png)

이 작업이 끝난 후에 녹색 글씨로 완료 되었다는 글이 보이실 겁니다. 만약 빨간색으로 Error 가 기술되어 있으면 에러 메세지를 확인 바랍니다.<br>
Once it has completed you'll see green text like below indicating that the work has been completed:

![StackWizard5](static/imgs/img7.png)

아래 처럼 S3 버킷 이름을 복사 해주세요. 앞으로 퍼스널라이즈를 위한 노트북의 실습시에 디폴트 버킷을 생성하는 로직으로 되어 있으나, 
지금 생성한 버킷으로 대체해서 사용하셔도 됩니다.<br>
Now that you have your environment created, you need to save the name of your S3 bucket for future use, you can find it by clicking on the `Outputs` tab and then looking for the resource `S3Bucket`, once you find it copy and paste it to a text file for the time being.

![StackWizard5](static/imgs/img8.png)



## Using the Notebooks

콘솔의 왼쪽 상단의 네비게이션 바에서 'Services'를 클릭 하신 후에 'SageMaker'를 찾으세요.<br>
Start by navigating to the SageMaker serivce page by clicking the `Services` link in the top navigation bar of the AWS console.

![StackWizard5](static/imgs/img9.png)

'SageMaker'를 입력하고, 서비스가 보이시면 클릭하세요. 
이후에 서비스 페이지로 부터 왼쪽 메뉴에 'Notebook Instances' 링크를 클릭 하세요.<br>
In the search field enter `SageMaker` and then click for the service when it appears, from the service page click the `Notebook Instances` link on the far left menu bar.

![StackWizard5](static/imgs/img10.png)

노트북 인스턴스의 끝에 'Open JupyterLab'을 클릭하세요.<br>
To get to the Jupyter interface, simply click `Open JupyterLab` on the far right next to your notebook instance.

![StackWizard5](static/imgs/img11.png)

이제 노트북의 가이드를 따라 실행 해주시고, 질문이 있으시면 랩을 진행하는 서포터 분이나, 아래 유튜브 비디오를 참조 해주세요.<br>
The rest of the lab will take place via the Jupyter notebooks, simply read each block before executing it and moving onto the next. If you have any questions about how to use the notebooks please ask your instructor or if you are working independently this is a pretty good video to get started:

https://www.youtube.com/watch?v=Gzun8PpyBCo

## After the Notebooks
작업이 끝나면, 앞에서 CloudFormation으로 생성한 stack을 지워야 합니다.다시 AWS 콘솔을 클릭하시고, 입력란에 'CloudFormation'을 입력하시고
메뉴가 보이시면 클릭하세료. <br>
Once you have completed all of the work in the Notebooks and have completed the cleanup steps there as well, the last thing to do is to delete the stack you created with CloudFormation. To do that, inside the AWS Console again click the `Services` link at the top, and this time enter in `CloudFormation` and click the link for it.

![StackWizard5](static/imgs/img9.png)

전에 생성한 demo stack 위에 'Delete' 버튼을 클릭 해주세요. <br>
Click the `Delete` button on the demo stack you created:

![StackWizard5](static/imgs/img13.png)

팝업 메뉴에 'Delete Stack'을 클릭 해주세요. <br>
Lastly click the `Delete Stack` button that shows up on the popup:

![StackWizard5](static/imgs/img14.png)

보이는 것럼 지워지고 있다는 것을 알 수 있을 겁니다. 'Delete Completed'가 보이면 모든 작업이 완료 된 것 입니다. 수고 하셨습니다. <br>
You'll now notice that the stack is in progress of being deleted. Once you see `Delete Completed` you know that everything has been deleted and you are 100% done with this lab.





