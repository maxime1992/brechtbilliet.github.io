---
layout: post
title: Creating a reactive calendar in Angular 4 and RXJS
published: false
author: brechtbilliet
comments: true
---

## Foreword

RxJS is an awesome library that helps us with creating **reactive web-applications**. Reactive web-applications can be overwhelming in the beginning but they can be really rewarding in the end.

This article is all about making that paradigm-switch from thinking imperative towards **thinking reactive**.
In this article we will learn how to write a reactive calendar-application in only a few lines of code.

We will use Angular, Angular material, typescript, RxJS and firebase as our main technology-stack, but article really focusses on reactive programming. Don't expect a deep dive in all RxJS operators, but rather expect how to draw, think and reason about reactive web-applications.

## The reactive calendar
This is the application that we are going to write.
![Reactive calendar](https://raw.githubusercontent.com/brechtbilliet/brechtbilliet.github.io/master/_posts/reactivecalendar/reactivecalendar1.png)
The reactive calendar is a small but complete (kindoff) calendar application that allows us to:

- Switch between different viewmodes:day, week, month.
- Navigate to previous and next days, weeks and months
- Add appointments, update appointments and remove them
- Search for specific appointments

The user can interact with the following UI-elements:

- **Next button:** Go to next day in Day-mode, week in Week-mode, etc...
- **Previous button:** Go to previous day in Day-mode, week in week-mode, etc...
- **Day, Week, Month buttons:** Change to viewmode
- **searchTerm input:** Filters the appointments live when we type in it
- **plus-buttons:** Allow the user to create new appointments
- **trashcan-buttons:** Allow the user to remove appointments
- **description inputs:** Allow the user to update the description of the appointment

**Note:** One small issue, we can only create appointments at noon =) But hey! Consider it some homework.


## Setting up the project

I've created a git branch to get us started. It contains the dumb components which are already implemented, it contains the smart component (without the reactive logic part), the setup and the styles. The goal ofcourse is to focus on the reactive part.

First of all, we will clone the project locally, and checkout the **initial** branch. This branch already contains all the uninteresting parts we don't want to write right now. 

### The component tree

![The component tree](https://raw.githubusercontent.com/brechtbilliet/brechtbilliet.github.io/master/_posts/reactivecalendar/reactivecalendar3.png)
The topbar-component, day-view-component, week-view-component, month-view-component and day-detail-component components are already written for us. In this article we will focus on the **app-root**. The smart component that will contain the reactive logic.


### Installing the project locally

In the terminal, go to the folder where you want to install the project and execute the following commands:

```
$ git clone git@github.com:brechtbilliet/reactive-calendar.git
$ cd reactive-calendar/reactive-calendar
$ git checkout initial
$ npm install
```

### Setting up firebase

We are using [firebase](https://firebase.google.com/) as our backend because it requires minimal setup. In these few steps we can complete the firebase configuration:

- Go to [https://firebase.google.com](https://firebase.google.com/) and click on the "GO TO CONSOLE" button and choose your google account.
- Click on the "Add project" button and choose a name for your project. Let's take **"reactive-calendar"** to keep it simple. 
- Click on the "CREATE PROJECT" button. Now we should be redirected to [something like this](https://console.firebase.google.com/project/reactive-calendar/overview).
- On the Authentication-tab, go to "SIGN-IN METHOD" and enable the "Anonymous" setting.
- Go back to the Overview by clicking on the home-symbol and then select "Add Firebase to your web app".
- Copy the config with the correct properties and replace the firebaseConfig object in src/app/app.module.ts with these properties.
This might look something like this:

```typescript
const firebaseConfig = {
    apiKey: "AIzaSyBuqjTJd5v6xTf8D2EZmvFUl8lseH8lVuHU",
    authDomain: "reactive-calendar.firebaseapp.com",
    databaseURL: "https://reactive-calendar.firebaseio.com",
    projectId: "reactive-calendar",
    storageBucket: "reactive-calendar.appspot.com",
    messagingSenderId: "3978123451455750"
};
```

Let's continue, start the project by running the following command and open your browser on [http://localhost:4200](http://localhost:4200). 

```
npm start
```

As you can see this just handles static data, the buttons/inputs won't work and the appointments are not loaded yet.
This is where we start from.

## Thinking reactive

This is what this article is all about, we are trying to forget imperative programming for now, and we are trying to evolve into a reactive mind-set.


### Imperative programming: What does the app have to do?

When we think about the functionality of our application, we quickly notice that there are quite a few cornercases. For every interaction the user makes with the UI, the app needs to handle this in a specific way. Sometimes it has to combine these actions together and handle them in a specific way as well. Take this crazy (but simple) example for instance.

<blockquote>
When the viewMode is changed to Week-mode, and the current viewMode is Month-mode, and the month was June, and the year was 2017, and an appointment was added, while the searchterm was set to "Brecht", then we would have to update...
</blockquote>

Yes, we would have to update a bunch of stuff. This is imperative thinking and it can become exhausting. There is a big chance that we forget certain corner-cases. Let's imagine that we have to combine that with asynchronous actions as well.

In the image below we can see all the different interactions the user might do in the calendar application.
![Application events](https://raw.githubusercontent.com/brechtbilliet/brechtbilliet.github.io/master/_posts/reactivecalendar/reactivecalendar2.png)

### Reactive programming: What data will change, and how will it impact the UI?
