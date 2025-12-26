---
cover:
  image: "images/jakob-owens-unsplash.jpg"
  # can also paste direct link from external site
  # ex. https://i.ibb.co/K0HVPBd/paper-mod-profilemode.png
  alt: "picture of a long asphalt road in the desert leading into the mountains"
  caption: "_Photo by: [Jakob Owens](https://unsplash.com/@jakobowens1) on [Unsplash](https://unsplash.com/)_"
  relative: false # To use relative path for cover image, used in hugo Page-bundles
date: 2024-01-04T00:00:00-00:00
description: "Fedora Silverblue"
tags: ["OSS", "Silverblue", "Fedora"]
categories: ["OSS", "Fedora"]
title: "Why did I choose Fedora Silverblue?"
---

Considering that the Operating System (OS) ecosystem has began to change its views on their core components and how they work over the years, I thought it would be a good idea for me to describe what brought me to the OS I use. To best put it simple, I chose [Fedora Silverblue](https://fedoraproject.org/atomic-desktops/silverblue/) because of its concept of “immutability” and being “atomic.” The terms immutability and atomic have come into play increasingly more over the past decade, and I feel like the idea of using an immutable or atomic OS deserves some attention.

The terms “immutable” or “atomic” means that everything at the base of the system is read-only. Meaning that everything outside of /etc and /var are the only directories that can be written to. All of the other directories cannot be tampered with. This makes OSes inherently more secure, because not other directory can be written to by malicious software, nor can they be modified by the user. If an update does not go as intended, the user can roll back the operating system upon the next boot.

Fedora Silverblue is what the Fedora Project calls an “Atomic Desktop.” This is because it is immutable in nature, and allows the user to roll back any changes that are made at the system level that might have gone wrong. I chose Silverblue because it keeps your system tidy, and prevents software from modifying core system files. All software is installed via Flapaks, in a Toolbx container, or via layering. They key term here is container. Toolbx allows you to create containers within a terminal that can run software just like a normal Fedora install, or even graphically outside of the terminal.

Silverblue both updates and layers software by utilizing rpm-ostree. By invoking this command, you can update the entire system and still have a point to roll back to if an update does go wrong. I always do ostree admin pin 0 before I do a major upgrade. This means that the most recent point that I’m on before invoking rpm-ostree rebase fedora:fedora/<…>/x86_64/silverblue By doing this, I will have that point to fallback on should the update go severely wrong.

The points made above represent my own reasons for choosing Fedora Silverblue, why did you choose the OS you’re on?
