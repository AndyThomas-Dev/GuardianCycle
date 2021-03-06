## 1. System Design
<p align="center">
<img src="media/architecture.png" alt="Overall systems architecture">
</p>
<p align="center">
  <i>
  Figure 1. Overview of GuardianCycle architecture
  </i>
</p>


### Contents
- [1. System Design](#1-system-design)
  - [Contents](#contents)
  - [a. Architecture of the entire system](#a-architecture-of-the-entire-system)
    - [Overview](#overview)
  - [b. Object-Oriented design of key sub-systems](#b-object-oriented-design-of-key-sub-systems)
    - [Desktop Application](#desktop-application)
    - [IoT Device](#iot-device)
    - [Web application](#web-application)
    - [Importance of object oriented design](#importance-of-object-oriented-design)
  - [c. Requirements of key sub-systems (in the form of selected user stories)](#c-requirements-of-key-sub-systems-in-the-form-of-selected-user-stories)
      - [User Story 1](#user-story-1)
      - [User Story 2](#user-story-2)
      - [User Story 3](#user-story-3)
      - [User Story 4](#user-story-4)
      - [User Story 5](#user-story-5)
    - [Requirements of key subsystems](#requirements-of-key-subsystems)
    - [Profile of users from stories](#profile-of-users-from-stories)
    - [Hierarchy of priorities](#hierarchy-of-priorities)
    - [IoT device](#iot-device-1)
    - [Desktop application](#desktop-application-1)
    - [Web application](#web-application-1)
  - [d. The evolution of UI wireframes for key sub-systems](#d-the-evolution-of-ui-wireframes-for-key-sub-systems)
    - [Paper Prototype – User Testing](#paper-prototype-%e2%80%93-user-testing)
    - [Desktop Application](#desktop-application-2)
    - [Web application](#web-application-2)
    - [Hardware development](#hardware-development)
    - [M5Stack UI development](#m5stack-ui-development)
    - [M5Stick UI development](#m5stick-ui-development)
  - [e. Details of the communication protocols in use](#e-details-of-the-communication-protocols-in-use)
    - [JavaScript Object Notation (JSON)](#javascript-object-notation-json)
      - [Update](#update)
      - [Route](#route)
    - [Mosquitto versus HiveMQ](#mosquitto-versus-hivemq)
  - [f. Details of the data persistence mechanisms in use](#f-details-of-the-data-persistence-mechanisms-in-use)
  - [g. Details of web technologies in use](#g-details-of-web-technologies-in-use)
  - [h. Reflective summary](#h-reflective-summary)

### a. Architecture of the entire system
#### Overview
GuardianCycle’s system architecture is composed of three different sub-systems: the Processing [desktop application](https://github.com/HumphreyCurtis/GuardianCycle/tree/master/desktop-application), the [web application](https://github.com/HumphreyCurtis/GuardianCycle/tree/master/web-application), and the [IoT device](https://github.com/HumphreyCurtis/GuardianCycle/tree/master/iot-device) (M5Stack and M5Stick). Each of these sub-systems is designed to fulfil three specific use cases and facilitate the full setup and use of the entire GuardianCycle system.

Firstly, the Processing-driven desktop application utilised a Google Maps Static API to track users' route information and journey analytics delivered via JSON data file of GPS points from the M5Stack. Secondly, the web application relayed tracking information to friends or family; and in the event of an incident, triggered by the IoT device, the emergency services.  Lastly, the M5Stack tracked users' route data, oriented an RGB LED indicator whilst both devices could send emergency GPS coordinates upon trigger in the form of a JSON string.

<p align="center">
<img src="media/SystemArchitecture.jpg" alt="Overview of System Architecture">
</p>
<p align="center">
  <i>
  Figure 2. Overview of system architecture 
  </i>
</p>

Three stakeholders, the user (cyclist), friends or family ie. 'Next of Kin' (desktop application) and emergency services (website), can interact with the GuardianCycle architecture at different stages. The IOT device is handled solely by the cyclist who is the primary user. The desktop application provides an interface for the user and family and friends to interact with the cyclist's data collected from journeys. It also allows users to note personal bests, goals and provide aesthetic data interaction. Lastly, the web interface is for the emergency services to track activity from GuardianCycle users within a region and denote if any emergency signals have been triggered at any coordinates, enabling rapid response and ultimately saving GuardianCycle users' lives. 

### b. Object-Oriented design of key sub-systems

From the outset, GuardianCycle endeavoured to take an object-oriented approach to the integration of key systems. This means that the various components that made up the system as a whole were considered as separate entities, each with their own set of encapsulated data and methods. Considerable attention was given to how those encapsulated entities interfaced with eachother, with a set of standard communication templates enforced throughout the project.

This was all the more important given our aims and ambition for the project. GuardianCycle strove to provide a whole suite of functionality to the user across a number of different platforms. This meant that the team as a whole needed to understand how the myriad parts interlocked. Taking this strong object-oriented approach fostered this understanding.

This UML use case diagram for this system (under the early working title of CyberDome) shows the number of different stakeholders involved and how they interact:

![UML Use Case Diagram](media/enduser.png)
<p align="center">
  <i>
  Figure 3. UML use case for GuardianCycle 
  </i>
</p>

The complexity of interactions between these functions was further explored in a UML class diagram which extended standard UML by attempting to overlay stakeholders over encapsulated methods and data:

![UML Class Venn diagram](media/initialuml.png)
<p align="center">
  <i>
  Figure 4. UML class venn diagram for GuardianCycle 
  </i>
</p>

This initial work proved the necessity of deciding at the outset the interfaces between these elements. Therefore it was an early design decision to ensure that all data passing between entities would belong to one of two classes, _Route_ or _Update_:

<p align="center">
<img src="https://github.com/HumphreyCurtis/GuardianCycle/blob/master/portfolio/media/route_update.png" alt="Route and Update classes">
</br>
<i>
  Figure 5. Route and Update classes 
  </i>
</p>

This ensured that all developers on the team understood exactly what data they could expect to send and receive from their individual components. 

#### Desktop Application

The [desktop application](https://github.com/HumphreyCurtis/GuardianCycle/tree/master/desktop-application) is intended to log and display cyclist's completed routes, providing them with statistics to track their performance including information about speed, distance and calories burned. The application was written using Processing and a UML class diagram corresponding to the source code is shown below:

![Processing UML Class diagram](media/processing_class_uml.svg)
<p align="center">
  <i>
  Figure 6. UML class diagram for desktop application
  </i>
</p>

This diagram shows that the Processing application follows the object oriented design philosophy, with each class representing a modular functionality, using a form of Model-View-Controller design pattern.  In brief the _Gui_ class serves as the View - producing the various on screen elements (buttons, etc) and displaying data from the route to the user.  

The Controller aspect is dually handled by _DataHandler_  and _MQTTHandler_ which is responsible for accepting input either from  the MQTT protocol or from the user and processing that accordingly. The Model element is handled by _Calculator_, _Maps_ and _PolyLineEncoder_ which take the JSON data from the _Route_ class (referred to above) and runs it through algorithms to determine calories, distance covered, etc and correspondingly places that data in geographical form (the _PolyLineEncoder_ acting to compress latitude / longitudinal data when sending over the lightweight MQTT network and to easily request data from the Google maps API).

#### IoT Device

The [IoT device](https://github.com/HumphreyCurtis/GuardianCycle/tree/master/iot-device), comprising of a M5Stack, M5Stick and LED matrix, was the heart of the GuardianCycle system: tracking the movement of the user, alerting other road users to the cyclists intentions, initiating emergency incidents and communicating data with other components of the system.

The M5Stack is a low-cost, lightweight, incredibly adaptable and internet-connected data collection device. Therefore, it was ideal for developing a GuardianCycle prototype. Despite these advantages the M5Stack did have some minor shortcomings. 

The lack of a GPS module located within the M5Stack meant that, for development purposes, all users' route data had to be hardcoded in the form of a JSON GPS coordinates string. Furthermore the short battery life of the M5Stack resulted in the LED using a considerable portion of the battery. These shortcomings are noted and mitigated in the future approach section.

The IOT device is envisaged as low-cost, low-energy, environmentally and user friendly, offering useful LED and GPS functionality. Collectively, we wanted the device to be durable, effective and safe for road use. Data was dually transmitted to either the [desktop](https://github.com/HumphreyCurtis/GuardianCycle/tree/master/desktop-application) or [web application](https://github.com/HumphreyCurtis/GuardianCycle/tree/master/web-application).  


<p align="center">
<img src="media/M5StateDiagram.jpg" alt="Overview of System Architecture">
</p>
<p align="center">
  <i>
  Figure 7. Full state diagram of M5Stack
  </i>
</p>

The above state diagram captures all of the states available to the user whilst using the M5Stack for cycling. The M5Stack is intended to be situated on the handlebars so that users can easily indicate left or right by pushing the corresponding button on the device. During and at the end of journey, the M5Stack will provide route coordinates as JSON data packages to either the website or desktop application. 

After turning GuardianCycle on, a suitable startup sequence utilising Spiffs library to display an image sequence begins the M5 functionality. At the start of their journey, the user is prompted to press the central button to begin. 

<p align="center">
<img src="media/proper-intro.gif" alt="Left turn gif">
</p>
<p align="center">
  <i>
  Figure 8. Start-up sequence for M5Stack
  </i>
</p>
 
<p align="center">
<img src="media/startjourney2.jpg" alt="Left turn gif" height=400>
</p>
<p align="center">
  <i>
  Figure 9. LED indicator prompting user to start journey
  </i>
</p>

After departure, the User will be presented with the M5Stack primary home screen. From this homescreen the user can use the Guardian cycles interface and buttons to:

* Render a route log drop down showing timer data. 
* Coordinate three different LED displays rendered by an LED matrix on the back of the helmet to provide optimal clarity to fellow road users.
* Terminate their journey and provide journey data to the desktop application. 
* Send an emergency signal to emergency services containting precise GPS coordinates of their current location. 

We believed that this indicator functionality would be incredibly useful to other road users and prevent cyclists from having to rely upon arm gestures. This will make road cycling easier and safer as the user would no longer have to physically indicate a turn especially if user is potentially [disabled](https://github.com/HumphreyCurtis/GuardianCycle/blob/master/portfolio/system-implementation.md#public-interest). The LED indicator feature is particularly useful for new or less confident cyclists. To enable the feature, the user pushes button 1 to make the indicator display a left arrow. The indicator will hold for 5 seconds; this allows enough time for the cyclist to turn safely:

<p align="center">
<img src="media/left2.jpg" alt="Left turn display" height=400>
</p>
<p align="center">
  <i>
  Figure 10. Button 1 press instantiating cyclist left turn 
  </i>
</p>

<p align="center">
<img src="media/left-turn.gif" alt="Left turn gif">
</p>
<p align="center">
  <i>
  Figure 11. LED indicator rendered on left turn 
  </i>
</p>

If button 3 is pressed, the LED will show a right arrow. 

<p align="center">
<img src="media/right-turn.jpg" alt="Right turn display">
</p>
<p align="center">
  <i>
  Figure 12. Display presented to cyclist on right turn 
  </i>
</p>
<p align="center">
<img src="media/right-turn.gif" alt="Right turn gif">
</p>
<p align="center">
  <i>
  Figure 13. LED indicator rendered on right turn 
  </i>
</p>

Finally, if button 1 is pressed and held for a duration of five seconds the LED and display will perform an emergency sequence and a JSON GPS data package via MQTT will be sent to the website for the emergency services to view and locate the cyclist's precise GPS position. Furthermore, the bright white light of the RGB LED would effectively notify fellow road users that the cyclist was in need of assistance. This is broadly similar to how warning lights on cars are used to notify fellow road users that a car has broken down. 

In the final production device this emergency function will be activated via gyroscope sensor should the cyclist be involved in a collision or a fall. Therefore, if the cyclist was unconscious or hurt this feature would be triggered automatically. Should the automatic trigger be a false positive then the cyclist will have a set period within which to cancel the incident, either via the M5Stick or M5Stack, prior to the incident message being dispatched.

<p align="center">
<img src="media/emergency.jpg" alt="Emergency light display">
</p>
<p align="center">
  <i>
  Figure 14. Display presented to cyclist in emergency scenario
  </i>
</p>

<p align="center">
<img src="media/emergency-light.gif" alt="Emergency light gif">
</p>
<p align="center">
  <i>
  Figure 15. LED rendered in emergency situation
  </i>
</p>

The M5Stack's second button can be lightly pushed to provide route logging timer functionality before returning to the primary homepage of the user interface. 

<p align="center">
<img src="media/route-log.jpg" alt="Emergency light display" height=400>
</p>
<p align="center">
  <i>
  Figure 16. Display presented to cyclist giving themroute-logging timer functionality
  </i>
</p>

The sending of end-of-route journey data is triggered by pressing button 2 which will deliver an encoded polyline or JSON file containing route data via MQTT to be parsed and rendered by the desktop application for the user. Furthermore, in emergency scenarios the same procedure is followed but data will instead be sent to the website for the emergency services to provide assistance. This publishing of user's coordinates and sending of JSON data packages from the M5Stack and stick via MQTT is displayed diagramatically as follows:

<p align="center">
<img src="media/StateAndLoopsOfStack.jpg" alt="State and loops of M5Stack and Stick">
</p>
<p align="center">
  <i>
  Figure 17. Loops, sequencing and MQTT publishing diagram of M5Stack and Stick
  </i>
</p>

The IoT device as well as being made up of the M5Stack and LED display also had a third element: a M5Stick. The M5Stick, due to its inherent capacity, had far less functionality than the M5Stack with this UML activity diagram describing its flow of activity:


<p align="center">
<img src="media/M5Stick-FSM-Updated.png" alt="M5 Stack Indicate">
</p>

<p align="center">
  <i>
  Figure 18. UML state diagram for MD5 Stick.
  </i>
</p>

As illustrated, the M5Stick can be used to alert the emergency services to an incident with two button presses - Arm and Alert - with positive confirmation needed to ensure the incident is raised, and an appropriate colour scheme (orange and red) to indicate escalation towards triggering the incident alert. Although this function is also available within the M5Stack it was considered beneficial to duplicate this functionality in the more portable M5Stick in case the cyclist became separated from their bicycle and was unable to reach it (for instance after a fall).

The IoT device devised by GuardianCycle therefore has a number of interlocking components: the M5Stick, the LED matrix and the M5Stack.  Object-oriented design was again key here, as it helped to ensure that they interacted with the rest of the system in a way that ensured encapsulation of data.  

#### Web application

There were two elements to the [web application](https://github.com/HumphreyCurtis/GuardianCycle/tree/master/web-application): friend-track-view and emergency-services-view. The former shared the current location of the cyclist with selected friends and family, the latter pinpointed the cyclist's location to the emergency services in the event of an incident.

A UML class diagram of the friend-track-view web application shows object oriented design in work. Admittedly this was a more lightweight approach, being a relatively simple JavaScript React application, than the Java driven Processing desktop application: 

![UML Class Diagram for friend-track-view](media/ftv_uml.png)
<p align="center">
  <i>
  Figure 19. UML class diagram for friend-track-view
  </i>
</p>

The use of the React framework (as discussed in [web technologies](#g-details-of-web-technologies-in-use) allowed object oriented thinking within the design.  As the above figure shows, components were used to encapsulate functionally different areas within the single page application.  _SignInSide_ and _Map_ were used for the two primary 'pages', while the _AlrtIncident_ class displayed details of location and triggered an alert of an emergency if neccesary.  Within all the components there are a number of discrete elements dedicated to serving different functions e.g. _RemoveMapLayer_, _componentDidMount_, etc.

#### Importance of object oriented design

The importance of the object oriented paradigm is potentially easiest to understand when looking at the activities of all three key sub-systems working together in harmony. Illustrated below in this UML activity state diagram, you can see a number of functions of the finished product (with gyroscope functionality currently on the development plan):

![Start to end UML Activity Diagram](media/whole_activity_uml.svg)

Within the diagram, white represents the IoT device, yellow the web application and blue the desktop application. Once the user starts tracking, the gyroscope immediately enters a loop whereby it looks for falls or collisions.  If one is detected then the user has an opportunity, using the M5Stick or M5Stack, to reset the alarm in case of a false-positive. If the user does not cancel via MQTT an Update JSON object is sent to both web applications (emergency-services-view and friend-track-view informing them of the incident) - this continues to loop as long as there is power in the device and the incident has not been cancelled.  If the gyroscope does not detect an incident then a location update is sent to friend-track-view only on a periodic basis.  This continues until the user stops tracking at which point they have an option to upload their route data, again via MQTT, to the desktop application where it can be viewed in Processing.  

A rigorous separation of data between these elements by using defined interfaces, e.g. object oriented design, is therefore necessary to manage this system effectively and safely considering the importance that users will place in it.

### c. Requirements of key sub-systems (in the form of selected user stories)

The user stories detail the many different interactions that the three stakeholders have within the GuardianCycle architecture. Designing user stories was an invaluable tool for understanding how stakeholders would effectively utilise the system and to demonstrate how the system should be properly used. Furthermore, user stories helped orient the device to succinctly determine which profile of users and target audience the device was best suited too. 

##### User Story 1

Dave is a keen cyclist who commutes to work everyday, and rides long distances at the weekend. He would like to keep track of his improving times and distances and share how he is progressing with friends and family. He is passionate about fitness and potentially wants to compete in an Iron Man competition next year. 

##### User Story 2

Hannah is trying to lead a healthier lifestyle and is starting to cycle occasionally. She would like to know how many calories she is burning by riding her bike and track her improvement. She hopes to lose weight and improve her lifestyle. 

##### User Story 3

Kevin is a mountain biker who is often alone for long periods of time in the hills. He is increasingly worried about his personal safety whilst mountain biking especially when the weather is poor and would like a way to get quick and direct help if there was an accident. 

##### User Story 4 

Mary is a mother of three, whose eldest son rides his bike to school everyday. She would like to know that her child is getting to school safely each day, and is concerned that other road users do not see him signalling when he is in traffic; their family home is located within a busy commuter belt. Furthermore, she would like to be able to track her child's location to make sure he gets to school and back home.

##### User Story 5

John is a cycling enthusiast and experienced first responder. He has worked for the emergency services for a number of years. In his local rural area has struggled to find an effective means to track down cyclists injured in the mountains and on the road. Particularly due to the lack of local landmarks and road signs.

#### Requirements of key subsystems

In light of these user stories, we used disciplined agile practices to prioritise the concerns and desires raised from the user stories as well as develop a clear profile of potential users of GuardianCycle. 

#### Profile of users from stories

<p align="center" >
<img src="media/ProfileOfUsers1.jpg" alt="Hierarchy of priorities" height=700>
</p>
<p align="center">
  <i>
  Figure 20. User story pool venn diagram
  </i>
</p>

These diagrams demostrate that a wide market section would be potentially interested in purchasing the GuardianCycle. In fact, users from disparate backgrounds such as non-cyclists expressed interest in GuardianCycle whilst of those users who frequently cycle, there was a range from mountain bikers to city cyclists and beginners to novices collectively interested in our product.

#### Hierarchy of priorities 

<p align="center" >
<img src="media/UserStories2.jpg" alt="Hierarchy of priorities" height=700>
</p>
<p align="center">
  <i>
  Figure 21. Six priorities for agile design process emphasised from user stories
  </i>
</p>

In response to these requirements we calibrated on certain key requirements for developing subsystems. 

#### IoT device

* The M5Stack had to be communicative and send necessary data to the desktop application and web application, enabling tracing functionality for GuardianCycle with JSON route data sent via MQTT. 
* The M5Stack had to fully render an RGB LED matrix to display the cyclists state (manoeuvres and emergency) to fellow road users - enabling complete safety as desired in [User Story 5](#user-story-5).
* The UI of the M5Stack had to be clean and clear preventing the cyclist from losing focus whilst cycling and dealing with needless distractions. 
* The M5Stack UI had to be intuitive for all age ranges, enabling any cyclist of any level of experience to effectively use the GuardianCycle, satisfying the wide user pool of potential users. 
* The route log functionality was developed to enable users to gain some cycling data to try and beat their own personal best times. This would satisfy user priorities for improvement. 
* The M5Stack had to detect incidents (using its internal gyroscope) and initiate an emergency event manually or automatically (gyroscope functionality not included in the minimum viable product).
* The M5Stick had to allow users to manually initiate an emergency event, or cancel an emergency event in the event of automatic detection.

#### Desktop application 

* Allow users to see information about the rides they have completed. Information must include information about speed, distance and the count of calories burned to fulfil [User Story 1](#user-story-1) and [User Story 2](#user-story-2). 

* Clearly show this information to the user in a clean and intuitive UI. Maps of previous routes were used to create a visually pleasing UI and provide more information about previous routes.

* Save all incoming route information so that users can restart the programme and still see all completed rides.

* Calculate all necessary route information from a set of coordinates sent from the M5Stack.

#### Web application 

* Provide an up to date and clear indication of a cyclist's location to selected friends and family (such as Mary in [User Story 4](#user-story-4)).  

* Differentiate to friends and family between normal activity and whether an incident had occurred (either detected automatically by gyroscope or manually initiated).  

* Alert friends and family by SMS or email in the event of an accident - functionality not included in the minimum viable product.  

* Ensure sensitive location data was only shared with authorised parties by the use of secure authentication.

* Provide up to date and clear indication of a cyclist's location to the emergency services in the event of an incident (as per John in [User Story 5](#user-story-5)) - noting that in the final product it is envisaged incident data being sent by the IoT device directly into systems owned and operated by emergency services control rooms.

### d. The evolution of UI wireframes for key sub-systems

#### Paper Prototype – User Testing

<p align="center">
  <img src="https://github.com/HumphreyCurtis/GuardianCycle/blob/master/portfolio/media/paper-prototype-gif.gif?raw=true">
</p>
<p align="center">
  A sample section of the paper prototype video.
</p>

An early part of the process saw the development of a paper-prototype, which was tested with several individuals. By using the paper prototype as a simulation for test users, this allowed us to quickly and efficiently test the key features of our design. 

Whilst the initial tester feedback was positive, in as much as the system was simple to use and understand, concerns were raised by one user who objected to having to provide email or other data that would enable them to be identified.

As a response, we agreed the finished product would not require a user to register before use. This would enable the fundamental road safety element – that of indicator/break lights to still be used. Unfortunately the GPS system required the website registration to be active, so the fall alert would not function in this user case.

Our user interface developed considerably from the initial paper prototype but retained some of the key features. The key features we retained include the journey analytics and the route viewer (albeit in a very different format).

The most notable difference is that the paper prototype was designed with a mobile application in mind (as evidenced by the phone screen background). However this specification evolved into a web application, a desktop application and an interface on the M5 devices, which all required a significant re-think in terms of design. 

#### Desktop Application

The key purpose of the desktop application is to display the previous routes the user has taken. The secondary purpose is to display information such as calories burned back to the user.

For the initial stages, the user interface was not a primary concern. Our main focus was ensuring the required functionality was attainable so initial wireframes simply involved one static map.

Once this was established, we merged this interface with the analytics function which would display the statistics for the displayed route.

We then decided we could be utilising the screen space more efficiently. Most notably, that we could fit multiple routes onto the page without it being overcrowded. At this stage, we experimented with a mulitude of different views to determine a best fit. This involved striking the right balance between displaying as much information as possible and the screen being too overcrowded. 

<p align="center">
  <img src="https://github.com/HumphreyCurtis/GuardianCycle/blob/master/portfolio/media/desktop-gui-wireframe2.png?raw=true" height="400">
</p>
<p align="center">
  Image: One of the early interface designs.
</p>

At the same time, we also wanted our design to be clear and intuitive so we kept the main features aligned and centralised. The font was also updated from the default font to a more modern font we felt was more in-line with our brand image.

<p align="center">
  <img src="https://github.com/HumphreyCurtis/GuardianCycle/blob/master/portfolio/media/final-ui-desktop.png?raw=true">
</p>
<p align="center">
  Image: Screenshot of final user interface.
</p>

Lastly, we realised we wanted to display more routes to the user and that the most practical way to do this was through the use of buttons to allow the user to scroll through a list of routes. By implementing this, the user could view an effectively unlimited number of routes by scrolling at their leisure.

#### Web application
In order to create the paper prototype, time was spent comparing a variety of online mapping products - Google Maps, OpenStreetMaps, OS Maps and the What's App Share Location specifically. Once a representative sample was obtained a prototype was mocked-up of what information the emergency services or a shared contact would wish to see.  A key requirement for this was a clean interface - neither user would wish to be presented with unnecessary or distracting information; the resulting prototype is shown below:

<p align="center">
  <img src="https://github.com/HumphreyCurtis/GuardianCycle/blob/master/portfolio/media/web-esv-prototype.jpg?raw=true">
</p>

The final version of our website maintained the key features of the paper prototype above. The user interface is minimalist with the key feature, the map, featuring prominently taking up the majority of the space on the page. This is in order to make the incident location as clear as possible for the user.

#### Hardware development 

Initially sufficient testing was performed on the capabilities of an LED matrix utilising an arduino. After which, the M5Stack was designed to perform the essential role as primary handlebar controller of the LED. The LED in this case a 5X5 RGB Matrix from Pimoroni connected to the M5Stack via GPIO pins. This is displayed in clear diagramatic form with connection on GPIO pins GND (Ground), SCL (Serial Clock), SDA (Serial Data) and 5V. 

<p align="center">
<img src="media/setup1.jpg" alt="Photo of M5stack Setup">
</p>
<p align="center">
  <i>
  Figure 22. Picture of M5Stack-LED setup whilst in development
  </i>
</p>

<p align="center">
<img src="media/M5StackLED.jpg" alt="Diagram of M5Stack setup">
</p>
<p align="center">
  <i>
  Figure 23. Diagram figure of M5 setup with 5x5 matrix
  </i>
</p>

#### M5Stack UI development 

The initial user interface designs for the M5Stack were basic, simply encoding button press functionality to change colours of orbs on the screen display. In order to gain familiarity with the M5Stack hardware.

<p align="center">
<img src="media/UI-development2.gif" alt="Emergency light gif">
</p>
<p align="center">
  <i>
  Figure 24. LED coloured orbs gaining familiarity with M5Stack hardware
  </i>
</p>

<p align="center">
<img src="media/UI-development.gif" alt="Emergency light gif">
</p>
<p align="center">
  <i>
  Figure 25. Initial prototyping of M5Stack UI 
  </i>
</p>

After this was acheived, clear colours were chosen for each command to enable clear visual clarity to the cyclist whilst in motion. Large arrows were programmed on the display and clear capital letters added to explain each maneuver: 

* Large blue arrow for left indicator signals.
* Large green arrow for right indicator signals.
* Red to denote emergency status has been triggered.

Having effectively programmed manoeuvres, development moved towards a press and hold feature to enable the triggering of an emergency situation for the cyclist. Collectively, it was decided that push and hold for 5 seconds to signal the declaring of an emergency was most effective to mitigate against mistaken pushes and triggering of an emergency. Particularly as the M5Stack's gyroscope had been difficult to function properly with the RGB 5x5 LED. 

Following this, we decided to present the user with a button press to initiate the start of the cycle for the sake of clarity. Followed by basic route log display on push of button 2 to provide the cyclist with journey progress data and the timings concerning progress made during their current cycle. As a later UI design change, for aesthetic purposes, it was decided to create a more professional startup sequence when the M5Stack was turned on - this helped make the device a more coherent and developed product. Particularly, typography was slightly modified from small and indiscernible writing to a clear coloured and eye-catching sequence.

<p align="center">
<img src="media/old-intro.gif" alt="Emergency light gif">
</p>
<p align="center">
  <i>
  Figure 26. Original intro sequence to M5Stack UI with indiscernable font
  </i>
</p>

Throughout the design process we tried to keep the Stack's UI clean, functional and simple. We considered this desirable as to prevent any forms of distraction for the cyclist - thus maintaining their eyes on the road at all times. This design feature has been adopted by numerous other applications - for instance, mobile phones automatically activating Do Not Disturb during [car journeys](https://support.apple.com/en-gb/HT208090).

#### M5Stick UI development 

<p align="center">
<img src="media/m5-stick.jpg" alt="M5 Stick image" height = 300>
</p>
<p align="center">
  <i>
  Figure 27. M5 Stick Intro UI 
  </i>
</p>

The small size of the screen and the limited input methods on the M5 stick was very limiting and we realised any complicated user interface loaded onto the stick would likely not be properly visible to the user.

<p align="center">
<img src="media/basic-stick-wireframe.png" alt="M5 Stick diagram image">
</p>
<p align="center">
  <i>
  Figure 28. Wireframes for M5 Stick UI
  </i>
</p>

With this in mind, a minimalist user interface was created that consisted simply of colours to indicate the state of the device and a short section of text indicating the state of the device.

### e. Details of the communication protocols in use

The IoT device communicated with the web and desktop application using the Message Queuing Telemetry Transport (MQTT) protocol. This is an open standard which typically runs over TCP/IP (Transmission Control Protocol / Internet Protocol) using a publish-subscribe model. MQTT was originally designed for monitoring oil pipelines in the desert and was therefore designed with a very lightweight footprint, in terms of battery requirements, bandwidth and code. It was these very qualities that made it perfect for a system designed to serve cyclists who typically cannot enjoy the luxuries of always on power and connectivity.

This diagram describes in essence how this communication worked:

<p align="center">
<img src="https://raw.githubusercontent.com/HumphreyCurtis/GuardianCycle/master/portfolio/media/Communication-Diagram.png" alt="Comms Diagram">
</p>

#### JavaScript Object Notation (JSON)

Over the MQTT protocol the IoT device communicated with the other components using JSON packages.  JSON was a good choice as a format as it is an open standard, human readable and lightweight, and was easy to integrate with all components in the system.

Two JSON structures were used, as detailed within [data communication](https://github.com/HumphreyCurtis/GuardianCycle/tree/master/data-communication), _Route_ and _Update_, examples of which follow.

##### Update

```json
{
   "lastCoord": [
      -2.5906169414520264,
      51.45326480293432
   ],
   "name": "John Doe",
   "timeSent": "2020-03-04T18:25:43.511Z",
   "isIncident": true
}
```
<p align="center">
Example of Update JSON
</p>

_Update_ variables consist of:

Variable | Type | Description
--- | --- | --- 
lastCoord |  int [ ] | Location geo-coordinates
name | String | Cyclist name
timeSent | Date | Date time group of last coordinate   
isIncident | boolean | Whether incident triggered either manually or via gyro


##### Route

```json
{
"time": 15,
"coordinates": [
          [
            -2.583932876586914,
            51.454240905300445
          ],
          [
            -2.585182785987854,
            51.45422084861252
          ],
          [
            -2.5838953256607056,
            51.454247590861115
          ]
	]
}
```

<p align="center">
Example of Route JSON
</p>

_Route_ variables consist of:

Variable | Type | Description
--- | --- | --- 
time | int | Duration of journey in minutes
coordinates | int [ ]  | Co-ordintates of journey route, recorded at preset frequency 

#### Mosquitto versus HiveMQ

To utilise the MQTT protocol a message broker is required.  A MQTT broker acts as a relay which receives messages from clients and then publishes those messages to appropriate clients which are subscribed to the same network. Initially the team were using the [HiveMQ](https://www.hivemq.com) MQTT broker for development efforts, primarily as it offered an easy to use MQTT Browser client which offered access to their public service.

However it soon became apparent that there were serious shortcomings with regards to the web application. Although when running on a development machine the web application flagged no issues, when it was uploaded to the internet the following warning would appear in the browser:

<p align="center">
<img src="https://github.com/HumphreyCurtis/GuardianCycle/blob/master/portfolio/media/web_blocked_content.png" alt="MQTT blocked content">
</p>
<p align="center">
  <i>
  Figure 29. Blocked content warning.
  </i>
</p>

Depending on the browser used this warning could be ignored and the page loaded normally - however clearly this was sub-par.

Investigation uncovered that the problem occurred because the browser was delivering content via Hypertext Transfer Protocol Secure (HTTPS), while accessing the MQTT network using an insecure method: WebSocket (WS) as opposed to WebSocket Secure (WSS). The attempt to access an insecure protocol (WS) within a protocol that guaranteed security (HTTPS) understandably triggered a warning. The warning was not triggered on a local machine as it was using HTTP (i.e. the non-secure version of HTTP) and likewise online when the warning was ignored, typically the browser would downgrade the connection to HTTP and allow access to the content.

The answer was clearly to use WSS within HTTPS, however the mechanism to do this was not readily understood.  Different versions of MQTT client libraries were trialled ([MQTT.js](https://github.com/mqttjs/MQTT.js) and [Eclipse](https://github.com/eclipse/paho.mqtt.javascript)) and the code within the web-application was extensively refactored in an attempt to fix the issue. After much investigation it transpired that the issue appeared to be the broker itself: switching to the [Mosquitto](https://mosquitto.org) MQTT broker resulted in a WSS connection over HTTPS and no warning. Mosquitto did not offer a web interface to their broker, but this was not an impediment as either the command line could be used or the HiveMQ browser client could continue to be used, but simply redirected at Mosquitto. Ultimately the production version of GuardianCycle would likely be using a private MQTT broker rather than a public one for reasons of security and performance.

At the end of the process the team felt that they had made considerable progress on understanding how MQTT can be integrated with a React single page application, and due to the dearth of information on this online, decided to contribute to wider community understanding by blogging a [concise tutorial](https://www.preciouschicken.com/blog/posts/a-taste-of-mqtt-in-react/) for others seeking to replicate this.

### f. Details of the data persistence mechanisms in use

The only data persistence mechanism used in the minimum viable product is a flat file [data repository](https://github.com/HumphreyCurtis/GuardianCycle/tree/master/data-repository) within the desktop application. This saves any incoming JSON MQTT messages to the application computer as JSON files, with a random string of numbers as the file name. When the application is loaded, the program looks through the data folder and reads every JSON file, creating a route object for each file. In the production version an online database would be used instead of flat files, so the information could be loaded by multiple devices. For this project it was enough to just save flat files as it mimicked the functionality of an online database. A local database could have been created instead of saving flat files, but the files are fairly simple, only containing two columns for coordinates and time. Therefore it made sense to save it simply as a JSON file and not over-engineer a solution by using a relational database.

For the website and M5Stack parts of the system no data persistence was implemented. In the production version of the system both of these components would likely have data persistence. The web application would require data persistence for instance for user authentication purposes. The IoT device may also use some form of persistence to store setup settings, and capture route information should upload or batteries fail - although clearly this would be balanced against the portability of the component.

### g. Details of web technologies in use

The [web application](https://github.com/HumphreyCurtis/GuardianCycle/tree/master/web-application) was built using the React web framework.  There were a number of reasons why this proved a good choice of framework: not least because React is extremely popular, topping StackOverflow's 2019 list for both most [Loved and Wanted Web Framework](https://insights.stackoverflow.com/survey/2019#technology-_-most-loved-dreaded-and-wanted-web-frameworks). Although popularity is not a positive in itself, the byproducts of having a vibrant ecosystem is - there are plentiful online resources and documentation, and when time comes to scale GuardianCycle it will not be difficult to find a pool of talent who can easily adapt to the technologies in use by the team.

Aside from its popularity, React also proved a good fit in other ways. As React uses a virtual Document Object Model (DOM) and a [reconciliation algorithm](https://reactjs.org/docs/reconciliation.html) it can update changes on page quickly without having to refresh the entire DOM. This speed is a key feature for a visualisation tool which is updating repeatedly to show a revised position. Also React being based around components allows re-use of code and makes it easy to impose a standard style, this fitted in very well with the object oriented design aspects of development; although admittedly this is likely something that would be used at scale - there was less need for this within a minimum viable product. Even so however within the current prototype use was made of [material-ui](https://material-ui.com) a library of web components which allows easy implementation of a standardised and professional user interface.

For the mapping solution within the web application [mapbox](https://www.mapbox.com) was selected. As well as being able to provide a mapping solution that was ideal for the site (clutter-free and easily understandable), they also featured excellent tutorials, compatibility with GeoJSON (one of the standards considered for use) and have a free pricing tier which was an important consideration prior to funding.

For connecting the website to the MQTT protocol, the [MQTT.js](https://github.com/mqttjs/MQTT.js) client library was used which is written in JavaScript and is intended for both node.js and the browser. Consideration was also given the to [Eclipse Paho](https://github.com/eclipse/paho.mqtt.javascript) JavaScript client - indeed the client was integrated successfully into the single page application when investigations were underway to determine the reason for a WebSocket Secure failure - which ultimately turned out not to be dependent on the client library. Eventually however MQTT.js was chosen due to its more extensive documentation and because its github repository was more actively maintained than its competitor.

Hosting was achieved using [Vercel](https://vercel.com) (formerly Zeit) which allows command line deployment of React/node.js web applications in a frictionless manner, and also features a free tier pricing model. Vercel also provides robust solutions and good documentation to ensure private API keys (which were required for mapbox) are not publicly viewable once deployed, which was a baseline security requirement.

### h. Reflective summary
The initial paper prototype allowed us to develop a basic understanding of what we wished to achieve as well as enabled us to perform some basic user testing. This initial design had to evolve in order to accommodate the limitations of the IoT device as well as of course the requirement for the project to be spread across multiple components and protocols. 

Once this had been refined, the creation of UML diagrams provided a more detailed overview of what we needed to build to ensure our project was a success. In addition, these diagrams helped us understand how the stakeholders would interact with the end product as well as ensure the entire team was on the same page in regards to what we would be producing.

This thorough understanding of objectives in turn allowed us to select appropriate technologies  such as MQTT, JSON and React, which each had specific advantages for the scenarios are device would face.

In summary, our design changed considerably from our initial paper prototype as we encountered challenges and added new features. In the course of this however all team members felt that this understanding and responding to challenges contributed greatly to our knowledge of software engineering; from the perspective of both the technologies employed and the practicalities of deploying them. 
