+++
title = "Clustering Address Reference Systems"
description = "Identifying types of street layouts using K-Means clustering."
date = "2019-04-30T00:00:23+01:00"
+++

*You can have a look at this [Jupyter notebook](https://github.com/shuaibiyy/address-interpolation/blob/master/Preparation.ipynb) for the data preparation, and [this one](https://github.com/shuaibiyy/address-interpolation/blob/master/ARS.ipynb) for the setup and implementation of the Address Reference System clustering.*

Say you're building search functionality similar to Google Maps search and you want your users to be able to search for addresses. A key part of delivering this would be collecting address data. Luckily, there are data sources like [Openaddresses](https://openaddresses.io/) and [Openstreetmap](https://www.openstreetmap.org/) that provide free access to such data. However, these data sources suffer from a problem of missing house addresses. Address data can be incomplete due to data recording errors, destruction or construction of buildings, or a myriad of other reasons. One way to handle missing addresses is to predict the coordinates of an address based on data about other addresses on the same street. Say you have the coordinates for *Main Street 1, 2, 3, & 5*; you could approximate the coordinates for *Main Street 4*. This is feasible because houses on a street tend to be laid out in some sequence, and the structure of that sequence is part of what is known as an **Address Reference System (ARS)**. In an ideal world, a city or municipality would have a standard ARS that's used for every street. In reality, that's often not the case.

![Arendsweg 13055, Berlin](https://rawgit.com/shuaibiyy/shuaib.me/master/themes/hugo-cactus-theme/images/ars/Arendsweg.png)

Addresses on a street follow some sequence and layout. For example, even numbered houses on one side of the road, and odd numbered houses on the other side, starting from the least number in the range to the highest. We can denote addresses on the right and left side of streets as *R* and *L* respectively, then a street can be described by a sequence such as *RRRLLL*, or *RLRLRLRL* etc. The street in the image above has its addresses in this sequence: *LLLLLLLLRLRLLLLLRRLRLRLLRRLLLLLLLLR*; odd numbered houses are on the left side of the street and the even numbered on the right side.

To improve the accuracy of address interpolation, we'd need to know the side of the street a house lies on; in other words, we need to know the ARS or layout of streets in a area. The approach I took was to apply the K-Means clustering algorithm to street parity sequences. The term parity refers the side of the road where an address is located i.e. left (*L*) or right (*R*). To extract address parities from streets, I used the [Pelias interpolation](https://github.com/pelias/interpolation) tool. You can have a look at the [data preparation notebook](https://github.com/shuaibiyy/address-interpolation/blob/master/Preparation.ipynb) for more details.

In order to apply the K-Means algorithm on street sequences, a little feature engineering is required. In this case that involves using scikit-learn's [CountVectorizer](https://scikit-learn.org/stable/modules/generated/sklearn.feature_extraction.text.CountVectorizer.html) on ngrams of length 5 for each street sequence parity. Keep in mind that the training data for this exercise is restricted to Berlin.

![CountVectorizer](https://rawgit.com/shuaibiyy/shuaib.me/master/themes/hugo-cactus-theme/images/ars/CountVectorizer.png)

Afterwards, we pass the vectors and value of *K* into the [K-Means algorithm](https://scikit-learn.org/stable/modules/generated/sklearn.cluster.MiniBatchKMeans.html), and get back cluster assignments for each street.

![K-Means](https://rawgit.com/shuaibiyy/shuaib.me/master/themes/hugo-cactus-theme/images/ars/KMeans.png)

Next, we can inspect the cluster assignments to see how well our algorithm did. Since we ran the K-Means with *K=2*, we expect that the algorithm learned 2 distinct types of street layouts. Let's inspect a couple of streets in **cluster 1**:

{{< figure src="https://rawgit.com/shuaibiyy/shuaib.me/master/themes/hugo-cactus-theme/images/ars/Winsstraße.png" caption="Winsstraße, Prenzlauer Berg, Berlin, Germany" >}}

{{< figure src="https://rawgit.com/shuaibiyy/shuaib.me/master/themes/hugo-cactus-theme/images/ars/Papelallee.png" caption="Pappelallee, Prenzlauer Berg, Berlin, Germany" >}}

And a couple of streets in **cluster 2**:

{{< figure src="https://rawgit.com/shuaibiyy/shuaib.me/master/themes/hugo-cactus-theme/images/ars/John-Schehr-Straße.png" caption="John-Schehr-Straße, Prenzlauer Berg, Berlin, Germany" >}}

{{< figure src="https://rawgit.com/shuaibiyy/shuaib.me/master/themes/hugo-cactus-theme/images/ars/Hufelandstraße.png" caption="Hufelandstraße, Prenzlauer Berg, Berlin, Germany" >}}

**What can we understand from the cluster assignments?**

Cluster 1 shows us streets with addresses in a **horseshoe** layout. The addresses run in sequence on one side of the street and wrap around to the other side.

Cluster 2 shows us streets with addresses in an **even-odd** layout. Even-numbered addresses are on one side of the street, while odd-numbered ones are on the other side.
