## Inspiration

The year 2020 brought changes all around the globe, imposing new standards when it comes to protecting everybody around you. In this time of need, one of the most harmed and overloaded institutions is the hospital, which is the first line of defense against the pandemic and a desperately needed place for some of us. People that have chronic, autoimmune or degenerative diseases must stay in touch with their doctors to keep their physical health in check.

As a multiple-sclerosis patient himself, our teammate Bogdan is bound to make regular visits to the hospital. The current global crisis complicated everybody's lives and his is no exception. He has to travel to other cities for periodic checks or to get treatment. It is frustrating to sometimes get back home with more questions than answers. However, Bogdan is a software engineer and he believes that AI could solve some of these issues.

## What it does

Q&AId solves this problem by:
* providing the user answers to questions on clinical data,
* providing the hospital with a transcript of what the patient needs, reducing the waiting time, and unloading the hospital triage.

<p align="center">
  <img align="center" src="https://github.com/medtorch/Q-Aid-Motivation/raw/master/misc/demo.gif">
</p>

Q&AId is a conversation agent that relies on a series of machine learning models to filter, label, and answer medical questions, based on a provided image as further described. The transcript can then be forwarded to the closest hospitals and the patient will be contacted by one of them to make an appointment.

Each hospital nearby has their models trained on private data that finetunes a visual question answering (VQA) model and other models, based on available data (e.g. brain anomaly segmentation). We aggregate all of the tasks that these hospitals can do into a single chat app, offering the user results and features from all nearby hospitals. When the chat ends, the transcript is forwarded to each hospital, a doctor being in charge of the final decision.

![Diagram flow](https://raw.githubusercontent.com/medtorch/Q-Aid-Motivation/master/misc/flow.png)

Q&Aid is simplifying the hospital logic backend by standardizing it to a Health Intel Provider (HIP). A HIP is a collection of models trained on local data that receives text and visual input, afterward filtering, labeling and feeding the data to the right models and generating at the end output for the aggregator. Any hospital is identified as a HIP holding custom models and labeling based on its knowledge.

## How we built it

There are three sections of the app that are worth mentioning:

### Q-Aid-App

* Created using React-Native.
* Authentication and database support by AWS Amplify.
* Awesome chat created using GiftedChat.
* Backed by the PyTorch core algorithms and models.

### Q-Aid-Core
* Server built with FastAPI
* DockerHub deployment as a docker image.
* script that partially builds the following diagram:

![backend_diagram](https://raw.githubusercontent.com/medtorch/Q-Aid-Core/master/aws_backend/architecture.png)

This stack builds up to:
* A DNS-Record for the application.
* An SSL/TLS certificate.
* An Application Load Balancer with that DNS record and certificate.
* An ECR Container Registry to push our Docker image to.
* An ECS Fargate Service to run our Q&Aid backend.

Inspired by this [tutorial](https://aws-blog.de/2020/03/building-a-fargate-based-container-app-with-cognito-authentication.html).

### Q-Aid-Models

* Visual Question Answering

Visual Question Answering is a challenging task for modern Machine Learning. It requires an AI system that can understand both text and language, such that it can answer text-based questions given the visual context (an image, CT scan, MRI scan, etc.).

Our VQA engine is based on MedVQA, a state-of-the-art model trained on medical images and questions, using Meta-Learning and a Convolutional Autoencoder for representation extraction, as presented [here](https://github.com/aioz-ai/MICCAI19-MedVQA/tree/c076f2cc174def26fa597fce4235b93f56658cc8).

* Medical Brain Segmentation

Medical segmentation is the task of highlighting a region or a set of regions with a specific property. While this task is mostly solved in the general-purpose setup, in the medical scene this task is quite hard because of the difficulty of the problem, humans having a bigger error rate when highlighting abnormalities in the brain and the lack of data.

Our model uses an [UNet](https://arxiv.org/pdf/1505.04597.pdf) architecture, a residual network based on downsampling and upsampling that has good performances on the localization of different features, as presented in the Pytorch [hub](https://pytorch.org/hub/mateuszbuda_brain-segmentation-pytorch_unet/), thanks to the work of Mateusz Buda.

* Medical Labeling

Medical labeling is the task of choosing what kind of image the user is feeding into the app. So far, possible labels are: brain, chest, breast, eyes, heart, elbow, forearm, hand, humerus, shoulder, wrist. Currently, our VQA model has support only for brain and chest, but we are working on adding support to multiple labels.

Our model uses a Densenet121 architecture from the [torchvision](https://pytorch.org/docs/stable/torchvision/models.html) module, the architecture having been proved suitable for medical imagery by projects like [MONAI](https://github.com/Project-MONAI/MONAI) that uses it extensively.

* Medical Filtering

Medical filtering is the task of labeling images in two sets, medical and non-medical, as we want to filter all non-medical data before being fed into the other machine learning models.

Our model uses a Densenet121 architecture from the [torchvision](https://pytorch.org/docs/stable/torchvision/models.html) module.

## Datasets

The datasets used in this project are the augmented version of:
* [VQA-RAD](https://www.nature.com/articles/sdata2018251#data-citations)
* [Tiny ImageNet](https://tiny-imagenet.herokuapp.com/)
* [Medical Decathlon](http://medicaldecathlon.com/)
* [Mednist](https://www.dropbox.com/s/5wwskxctvcxiuea/MedNIST.tar.gz) - the dataset is kindly made available by [Dr. Bradley J. Erickson M.D., Ph.D.](https://www.mayo.edu/research/labs/radiology-informatics/overview) (Department of Radiology, Mayo Clinic) under the Creative Commons [CC BY-SA 4.0 license](https://creativecommons.org/licenses/by-sa/4.0/).


## Challenges we ran into
The hackathon has been quite a journey for the past few months, as the idea has constantly evolved.

At first, Tudor came up with the idea that PyTorch would need an Explainable Artificial Intelligence module. We decided that we want support for module interpretability and Tensorboard support, we called it TorchXAI. We learned a lot about model interpretability and how to integrate features in Tensorboard from the PyTorch API as plugins, Bogdan implemented a ton of algorithms. After a few weeks, Andrei showed us [captum](https://captum.ai/), which did all that we wanted to do, but better.

A bit demotivated, we were searching for a new idea, Bogdan came up with a medical use case, to use PyTorch to enhance hospitals. After Bogdan motivated us to continue, we started shaping the new idea, to find use cases, figure out the needs, what can be done and, most importantly, what we could do.

As a multiple-sclerosis patient himself, Bogdan is bound to make regular visits to the hospital.  The current global crisis complicated everybody's lives, and MS patients are no exception. Some of them have to travel to other cities for periodic checks or the treatment, and it can be frustrating and time-wasting. But he believed that AI could solve some of the issues and motivated us that we can solve this together.

After we found a good problem to work on, we started giving each other continuous feedback, working on different ideas. Tudor and Bogdan are contributors in the OpenMined community, a community that works to enable privacy-preserving machine learning. We expect that a future release will enable us to do Federated Learning at scale in the cloud. We decided that the hackathon would be the best place to start working on applying machine learning in healthcare and afterward the right next steps would be to enable privacy in healthcare so that hospitals could exchange more data.

Afterward, Bogdan came up with the notion of Health Intel Provider - HIP. This would be an abstraction that we would use for any hospital, research lab, or just a medical data owner that would want to join our network to train algorithms on any task. This would become the computational backbone required to use any kind of machine learning or privacy tools in a hospital.

At this moment, we decided that we want a medical chat that can help interpret medical imaging and define medical terms for the user as well as provide medical transcripts for the doctors to understand the needs of the patient.

After that, the data search started. After we saw how little data is available publicly in healthcare, we realized that we are on the right track and our work could have a good impact. After finding the right datasets and use cases, Tudor and Andrei started to train the models. Andrei came up with the idea of using a Visual Question Answering model because it would fit well with our medical chat task. Down the road, we faced tons of bugs, from the incompatibility of react-native with TorchScript to adapting different data distributions to match each other, but the most important thing is that we've learned a lot and had fun doing it.

At this point, we are looking forward to integrating more models (the next one being on retinopathy) and privacy tools to enable private location sharing and maybe even inference on private data by using PySyft.

## Accomplishments that we're proud of

### Team

Tackling a difficult problem that is important to us and being able to deliver a working proof of concept solution is a great source of pride for our team. This feat involves building and integrating several distinct moving parts ranging from machine learning pipelines to cloud infrastructure and mobile development. It requires a good understanding of all systems involved and, above all else, great communication, prioritization, and scheduling within the team. We view having successfully navigated all of the above in a relatively short period of time as a significant accomplishment.  

### Bogdan

I am really happy that we continuously found new sources of motivation to create a tool that might tackle some real issues, and that we learned a lot on the way. And I hope we made a small step in the right direction.

### Tudor

This hackathon has been a source of valuable lessons and great achievements, the ones that I am most proud of are: made progress towards solving a real-world hard problem, managed to sharpen my communication and computer vision skills, made my first technical project with my older brother. <3

### Andrei

The PyTorch Summer Hackaton was a way for me to explore ideas outside of my typical areas of interest. Deep Learning for Medical applications has a lot of different issues compared to the more traditional Deep Learning tasks. I enjoyed learning about the solutions addressing the lack of qualitative data and the network architectures.
This field has the potential of having a great impact on AI for Social Good, and I'm glad that we were able to develop a working prototype showcasing a Deep-Learning enabled medical assistant. Such tools could change the medical landscape, providing access to powerful diagnosis tools even in the most remote corners of the world.


### Horia

As a student, solving clearly defined problems using known methods is the norm. As such, having the opportunity to identify a meaningful problem and explore novelty solutions offers great challenges and satisfactions. I am tremendously proud to be part of an effort that democratizes access to quality diagnoses while preserving the essence of the doctor-patient relationship. AI will shape the future of medical technology and nudging this transformation is a true personal accomplishment.

## What we learned

As a team, we've learned how to express our ideas the right way and how to give constructive feedback. We've learned that we are here for the journey and we should make the most out of it, even if there are different opinions along the road.

As individuals, each of us learned valuable social and technical lessons and even brand new skills from scratch. Here are a few of them:

### Bogdan learned:
* XAI and model interpretability
* how to make conversational agents
* react-native
* how to deploy on AWS

### Tudor learned:
* TorchScript
* FastAPI
* Torch Hub
* MONAI

### Andrei learned:
* VQA
* segmentation
* captum

### Horia learned:
* PyTorch
* computer vision
* presentation skills


## What's next for Q&Aid

Q&Aid has three tracks for its future:
* Integrating more medical machine learning models.
* Adding more features to the app.
* Integrating OpenMined technologies for privacy.
* Recruiting medical experts, doctors, and patients.


Andrei and Tudor are looking into the first track, integrating retinopathy detection and generative models as well for augmentation.

Bogdan is working on the second track, integrating better authentification services, searching for better distributed architectures and polishing the application UI based on feedback.

Bogdan and Tudor are working on applying Federated Learning between HIPs using PySyft from OpenMined, a library that wraps PyTorch and other data science tools for private model sharing and training. Both of them are working as active contributors in OpenMined to bring these privacy features closer to the healthcare scene.

Horia is working on the fourth track, searching for motivated students at the medical faculty in Bucharest that could help us gather data to furthermore train our VQA model and give feedback on its answers.
