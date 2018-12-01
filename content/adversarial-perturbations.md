+++
title = "Adversarial Perturbations"
description = "Fooling Deep Learning Models"
date = "2018-12-01T00:12:23+01:00"
+++

Adversarial perturbations are special kinds of distortions that trick machine learning and deep learning models into misclassifying their input. Adversarial examples in neural networks were first discovered and explored by Szegedy et al. (2013) in their paper: [Intriguing properties of neural networks](https://arxiv.org/abs/1312.6199). They showed that distortions or perturbations can be learnt from a model, which when applied to its input, cause the model to confidently misclassify the input.

{{< figure src="https://rawgit.com/shuaibiyy/shuaib.me/master/themes/hugo-cactus-theme/images/ostrich.jpeg" caption="Take a correctly classified image (left image in both columns), and add a tiny distortion (middle) to fool the ConvNet with the resulting image (right). Taken from http://karpathy.github.io/2015/03/30/breaking-convnets/." >}}

Adversarial examples are input that have been perturbed. Adversarial training is a technique for improving the defense of a deep learning model against adversarial perturbations by incorporating adversarial examples during model training.

Recently, as part of my academic coursework, I chose to review a paper titled: [DeepFool: a simple and accurate method to fool deep neural networks](https://arxiv.org/abs/1511.04599). The crux of the paper is the introduction of an algorithm called DeepFool that is effective at finding the minimal perturbations necessary to fool DNNs. Furthermore, the perturbations that are learnt using DeepFool are more effective for adversarial training compared to a pre-existing method called Fast Gradient Sign Method (FGSM). In order to get up to speed with adversarial learning, I found these sources especially helpful:

* [Tricking Neural Networks: Create your own Adversarial Examples](https://ml.berkeley.edu/blog/2018/01/10/adversarial-examples/)
* [Lecture 16 | Adversarial Examples and Adversarial Training
 By Ian Goodfellow](https://www.youtube.com/watch?v=CIfsB_EYsVI)
 * [Breaking Linear Classifiers on ImageNet](http://karpathy.github.io/2015/03/30/breaking-convnets/)
* [Intriguing properties of neural networks](https://arxiv.org/abs/1312.6199)
* [Explaining and Harnessing Adversarial Examples](https://arxiv.org/abs/1412.6572)
* [Universal adversarial perturbations](https://arxiv.org/abs/1610.08401)
* [Foolbox: A Python toolbox to benchmark the robustness of machine learning models](https://arxiv.org/abs/1707.04131)
