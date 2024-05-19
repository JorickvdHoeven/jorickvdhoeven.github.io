---
title: "Sensor network transfer methods"
excerpt: "Researching how to most efficiently pull information from a distributed sensor network onto a high-powered mobile sink to allow reserachers to collect data without needing collect sensors or visit each sensor."
imgae: "oxford-sensenet.png"
# sidebar:
#   - title: "Role"
#     image: "oxford-banner.png"
#     image_alt: "oxford logo"
#     text: "Masters student, Research"
#   - title: "Responsibilities"
#     text: "Examining the effects of movement on data collection protocols in a sensor network and building the fastest possible transfer protocol to pull data from the network into the sink."
---

In the final year of my studies at the University of Oxford I joined a research group working on a sensor network. My work consisted in investigating reliable transfer methods between the nodes of the sensor network and a mobile sink which would travel through the network and gather all of the data in the network wirelessly. If you are interested in my research I have posted my thesis below.

## Abstract

This project investigates the use of mobile sinks in wireless sensor networks.It focuses on three different transfer methods - Single Acknowledgement, Double Buffered and Sliding Window. The Single Acknowledgement transfer method is a naive method which acknowledges every packet it receives. The Double Buffered method keeps the same transfer protocol but reduces transfer times by performing time intensive data preparation in advance. The Sliding Window method is a variation of the TCP cumulative acknowledgement sliding window which only uses one timer. The project also proposes a way to select with which node a mobile sink is likely to have the longest connectivity based on several RSSI values taken over a period of time. All of the methods presented in this project are designed to be integrated into the WildSensing project which focuses on monitoring badgers in the woods around Oxford.

[read more...](mobile-sensor-network-data-sinks.pdf)
