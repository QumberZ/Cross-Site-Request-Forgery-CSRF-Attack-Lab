# Cross-Site-Request-Forgery-CSRF-Attack-Lab
This lab has been tested on our pre-built Ubuntu 20.04 VM, which can be downloaded from the SEED website. Since we use containers to set up the lab environment, this lab does not depend much on the SEED VM. You can do this lab using other VMs, physical machines, or VMs on the cloud. 
Qumber Zaidi
CSC 223 Lab Report

Cross-Site Request Forgery (CSRF) Attack Lab

To get started, we first go to the seed labs website, https://seedsecuritylabs.org/Labs_20.04/Web/Web_CSRF_Elgg/  and we download the labset up files and unzip them. Then we open up the command line or terminal and switch to the Labsetup directory and run a few commands. We run Docker-compose build, which creates all of the containers in order for us to use the websites and do this lab. Then we run docker-compose up which starts all of the containers and gets everything up and running. We can test to see if these containers are running using the command, docker ps. This command shows the container id, image, ports, status, name, and other information. Before testing the URLs on the web browser, we need to edit the /etc/hosts files to add the URLs next to their IP addresses. We can do this by running sudo gedit /etc/hosts, or sudo nano /etc/hosts, depending whether your OS has gedit installed or not. Then we can test the url by putting any of the following URLs we use for this lab into the web browser. If we see the login page for www.seed-server.com, then we know the containers are running. We log in and continue the next steps for the lab using the admins created for us already.

3.1 Task 1: Observing HTTP Request. In Cross-Site Request Forget attacks, we need to forge HTTP requests. Therefore, we need to know what a legitimate HTTP request looks like and what parameters it uses, etc. We can use a Firefox add-on called "HTTP Header Live" for this purpose. The goal of this task is to get familiar with this tool. Instructions on how to use this tool is given in the Guideline section (§ 5.1). Please use this tool to capture an HTTP GET request and an HTTP POST request in Elgg. In your report, please identify the parameters used in this these requests, if any. 

For this task, we need to observe HTTP Requests that our browser sends out everytime we click something on the UI. Each functioning feature or button, will always send out HTTP requests with information that we can access. We can access this using the FireFox extension or add-on called "HTTP Header Live”. Once you are able to set that up, you can have that open on the side view and click on any button or any click that would capture a HTTP request and view the response on the tool. For example, figure below, I clicked on the add button to add Alice as a friend using Samy’s account. And the requests below are what were captured. The parameters used in the request are ?friend=56&__elgg_ts=1665973772&__elgg_token=PCtIngK2Qc8A-5sch3zpRQ&__elgg_ts=1665973772&__elgg_token=PCtIngK2Qc8A-5sch3zpRQ
 

 

3.2 Task 2: CSRF Attack using GET Request In this task, we need two people in the Elgg social network: Alice and Samy. Samy wants to become a friend to Alice, but Alice refuses to add him to her Elgg friend list. Samy decides to use the CSRF attack to achieve his goal. He sends Alice an URL (via an email or a posting in Elgg); Alice, curious about it, clicks on the URL, which leads her to Samy’s web site: www.attacker32.com. Pretend that you are Samy, describe how you can construct the content of the web page, so as soon as Alice visits the web page, Samy is added to the friend list of Alice (assuming Alice has an active session with Elgg). To add a friend to the victim, we need to identify what the legitimate Add-Friend HTTP request (a GET request) looks like. We can use the "HTTP Header Live" Tool to do the investigation. In this task, you are not allowed to write JavaScript code to launch the CSRF attack. Your job is to make the attack successful as soon as Alice visits the web page, without even making any click on the page (hint: you can use the img tag, which automatically triggers an HTTP GET request). Elgg has implemented a countermeasure to defend against CSRF attacks. In Add-Friend HTTP requests, you may notice that each request includes two weird-looking parameters, elgg ts and elgg token. These parameters are used by the countermeasure, so if they do not contain correct values, the request will not be accepted by Elgg. We have disabled the countermeasure for this lab, so there is no need to include these two parameters in the forged requests. 

For the second task, we want the hacker(samy) to have the victim(alice) added as a friend by a posting on Elgg. This link should lead Alice to the attacker website which would trigger an HTTP response and would make Samy Alice’s friend against her own will. First, we open the HTTP Header Live tool and click on the Add Friend button on Alice’s profile from Samy’s account. This would then trigger a POST response and would list the information for the HTTP request. You would be able to see the attributes and parameters for the request. We would be able to access the get request and parameters of Alice’s Guid using the parameters provided in the URL. We take the link and place it somewhere safe, where we would be able to access it again. We then go to Samy’s profile and view the source code. Using the source code, we find the GUID for Samy which is 59. 
 


http://www.seed-server.com/action/friends/add?friend=56

Samy guid= 59
alice guid=56


{"user":{"guid":59,"type":"user","subtype":"user","owner_guid":59,"container_guid":0,"time_created":"2020-04-26T15:23:51-04:00","time_updated":"2020-04-26T15:23:51-04:00",

We then place Samy’s GUID instead of Alice’s for the clickable link later on. http://www.seed-server.com/action/friends/add?friend=59


 
We want to also change this for the img src in the addfriend.html file so we also can’t copy and paste in nano to do this. So we have to copy the contents of the html file in a separate text editor and update the code there. Once we do this, we need to apply these changes to the actual html file. So, we run the docker cp addfriend.html command and paste the directory we were in for the docker shell for the attacker container.

 docker cp addfriend.html 9c9bcaacee69:/var/www/attacker/

To check if the changes applied, we go back to the other terminal tab for the attacker docker container and run the cat addfriend.html command. If we see the contents of the html file match our updated file, we know that changes have applied.  

Now let's test our changes. We open www.attacker32.com and click the dd-Friend Attack link. Once we do that, we see that it creates a forged HTTP GET request to Alice’ s profile to have Samy added as a friend. We can also check the source page and see if the code matches the code from the html file.
 
Now, if we go to Alice’s friends, we see that Samy is now one of them. This is because the Link we clicked had forged a HTTP GET request which added Samy as a friend using his unique GUID.
 

So this was an attack using a GET Request.


3.3 Task 3: CSRF Attack using POST Request After adding himself to Alice’s friend list, Samy wants to do something more. He wants Alice to say “Samy is my Hero” in her profile, so everybody knows about that. Alice does not like Samy, let alone putting that statement in her profile. Samy plans to use a CSRF attack to achieve that goal. That is the purpose of this task. One way to do the attack is to post a message to Alice’s Elgg account, hoping that Alice will click the URL inside the message. This URL will lead Alice to your (i.e., Samy’s) malicious web site www. attacker32.com, where you can launch the CSRF attack. The objective of your attack is to modify the victim’s profile. In particular, the attacker needs to forge a request to modify the profile information of the victim user of Elgg. Allowing users to modify their profiles is a feature of Elgg. If users want to modify their profiles, they go to the profile page of Elgg, fill out a form, and then submit the form—sending a POST request—to the server-side script /profile/edit.php, which processes the request and does the profile modification. The server-side script edit.php accepts both GET and POST requests, so you can use the same trick as that in Task 1 to achieve the attack. However, in this task, you are required to use the POST request. Namely, attackers (you) need to forge an HTTP POST request from the victim’s browser, when the victim is visiting their malicious site. Attackers need to know the structure of such a request. You can observe the structure of the request, i.e., the parameters of the request, by making some modifications to the profile and monitoring the request using the "HTTP Header Live" tool. You 
may see something similar to the following. Unlike HTTP GET requests, which append parameters to the URL strings, the parameters of HTTP POST requests are included in the HTTP message body (see the contents between the two * symbols):

For this task, we want to attack the victim again but this time using the POST method to post a statement on the victims page without their consent. We want to edit the editprofile.html file so we are able to forge a POST request to trigger the attack. We open up the file using a text editor and view the contents of it to see what we can change. We head over to Samy’s profile and click edit profile. Then in the About me section, we type “Samy is my hero.” and do the same for the brief description. Before saving this updated profile, we open the HTTP Header Live tool to record the HTTP requests that are being captured. We intend to utilize a CSRF attack.
 
We see that the guid is 59 and we want to change it to 56 for Alice so we can process the attack.
We copy the Post request,  http://www.seed-server.com/action/profile/edit , and want to modify the Edit Profile file. So we head over to the text editor where we have the contents of the file open, and we replace the 
 p.action = "http://www.example.com";
With p.action = "http://www.seed-server.com/action/profile/edit";
under the construct the form section.

Where it says,  fields += "<input type='hidden' name='name' value='****'>";
We place it with,  fields += "<input type='hidden' name='name' value='Alice'>";


Then we view the brief description from the HTTP request 
_elgg_token=VLOdsNoEUAmmXEr4FBmsSA&__elgg_ts=1666022134&name=Samy&description=<p>Samy is my hero.</p> &accesslevel[description]=2&briefdescription=Samy is my hero&accesslevel[briefdescription]=2&location=&accesslevel[location]=2&interests=&accesslevel[interests]=2&skills=&accesslevel[skills]=2&contactemail=&accesslevel[contactemail]=2&phone=&accesslevel[phone]=2&mobile=&accesslevel[mobile]=2&website=&accesslevel[website]=2&twitter=&accesslevel[twitter]=2&guid=59

And paste the “Samy is my hero” to the brief description to the editprofile.html file.

fields += "<input type='hidden' name='briefdescription' value='Samy is my hero'>";

Then we change the guid to the victim as a target. So since we are targeting Alice, we place 56 as the guid.
 
Then we save our changes to the file. We then run the command, docker cp editprofile.html 9c9bcaacee69:/var/www/attacker/  to copy the contents of the file to the attacker container.


 
 Once we see our modification to the file, then we know it was successfully changed and copied.
Then we log out of Samy’s account and log into Alice to proceed with the attack. We see that Alice’s profile is empty. Now we go on the www.attacker32.com site and click on the Edit-profile Attack. Once we do that, we refresh Alice’s profile and the description pops up on her page. This means the attack successfully went through.
 

 Question 1: The forged HTTP request needs Alice’s user id (guid) to work properly. If Boby targets Alice specifically, before the attack, he can find ways to get Alice’s user id. Boby does not know Alice’s Elgg password, so he cannot log into Alice’s account to get the information. Please describe how Boby can solve this problem. 

Boby can solve this problem by obtaining Alice’s GUID from Alice’s profile and viewing the source page. Then Boby can search for the GUId directly from the contents of the source code. Another way he can get the GUId is by using the HTTP Header tool and clicking the add friend button to trigger an HTTP request. The tool would capture the request and give information which includes the GUID.

Question 2: If Boby would like to launch the attack to anybody who visits his malicious web page. In this case, he does not know who is visiting the web page beforehand. Can he still launch the CSRF attack to modify the victim’s Elgg profile? Please explain

If Boby does not know the GUID for the specific person, then he cant launch the CSRF attack to modify the victim’s Elgg profile. The reason is, Boby would need the GUID of the victim as well as the name of the victim as he would need to fill in the values within the contents of the html file.

4. Lab Tasks: Defense CSRF is not difficult to defend against. Initially, most applications put a secret token in their pages, and by checking whether the token is present in the request or not, they can tell whether a request is a same-site request or a cross-site request. This is called secret token approach. More recently, most browsers have implemented a mechanism called SameSite cookie, which is intended to simplify the implementation of CSRF countermeasures. We will conduct experiments on both methods. 

4.1 Task 4: Enabling Elgg’s Countermeasure To defend against CSRF attacks, web applications can embed a secret token in their pages. All the requests coming from these pages must carry this token, or they will be considered as a cross-site request, and will not have the same privilege as the same-site requests. Attackers will not be able to get this secret token, so their requests are easily identified as cross-site requests. Elgg uses this secret-token approach as its built-in countermeasures to defend against CSRF attacks. We have disabled the countermeasures to make the attack work. Elgg embeds two parameters elgg ts and elgg token in the request. The two parameters are added to the HTTP message body for the POST requests and to the URL string for the HTTP GET requests. The server will validate them before processing a request. 

This part of the lab demonstrates how to prevent these attacks. To prevent CSRF attacks, web applications may integrate a secret token into their pages. To be considered a cross-site request and to receive the same rights as requests coming from the same site, all requests from these sites must contain this token. Otherwise, they will be regarded as incomplete. The attacker's searches will be marked as cross-site requests because they won't be able to get this secret token.

To get started, we open up a new tab in the terminal and switch to the shell of the defense container, which is the elgg container by entering the docker container id after docksh. 
docksh 74b9250cd2d7

Then we turn on the countermeasure. We change to the directory of the countermeasure provided from the lab instructions. 
cd /var/www/elgg/vendor/elgg/elgg/engine/classes/Elgg/Security 
Then, we want to modify and comment out the line that disables the countermeasure so it becomes enabled again. We do this by running nano Csrf.php

 

We scroll down to the line we want to comment out, then save and exit.

 
To see if the changes were saved, we run the cat command to view the file in the shell.


 
We have successfully enabled the countermeasure. To continue on with this task, we must remove all descriptions and friends added from Alice’s profile. It should look like how it did before we did any of the attacks. To test out the countermeasure, we use the attacker32 site to forge HTTP requests to GET and POST a description and add Samy as a friend the same way we did for task 2 and 3. If doing so does not work, then the countermeasure is doing its work successfully! This defends against any of the CSRF attacks.



 
 
 
 You should also get an error alert saying “form is missing token or fields” when you try to send out an attack. 

 


 





Task 5: Experimenting with the SameSite Cookie Method 
Most browsers have now implemented a mechanism called SameSite cookie, which is a property associated with cookies. When sending out requests, browsers will check this property, and decide whether to attach the cookie in a cross-site request.

Link A

When we click on link A, we are shown 3 options for the SameSite Cookie experiment
A. Sending Get Request (link)
http://www.example32.com/showcookies.php
B. Sending Get Request (form)
C. Sending Post Request (form)




When experimenting with the GET and POST requests, it displayed all of the cookies like cookie-normal, cookie-lax, cookie-strict for both of the requests. This is because the request is a samsite request.
 


 



The SameSite element allows you to specify whether your cookie should only be used for first-party or same-site purposes.

LINK B
SameSite Cookie Experiment
A. Sending Get Request (link)
http://www.example32.com/showcookies.php
B. Sending Get Request (form)
C. Sending Post Request (form)
 
 
Testing with the GET and POST requests, it is observed that a cross-site cookie is established. A cross site cookie allows a site attacker to establish a cookie for a browser on another site server’s cookie domain. A malicious site can use the cookies of another site and create attacks using it.



Please describe what you see and explain why some cookies are not sent in certain scenarios.

Link A
The same-site request, all cookies (cookie-normal, cookie-lax, and cookie-strict) were displayed when we performed GET and POST queries. 

Link B
There are distinctions between GET requests and POST requests during a cross site request. For the GET request, cookies of the types (cookie-normal, cookie-lax) will be displayed, however for the POST request, cookies of the type (cookie-normal) will be displayed because the request is cross-site.



 • Based on your understanding, please describe how the SameSite cookies can help a server detect whether a request is a cross-site or same-site request.

Cookies that are connected to all requests to a certain origin, regardless of who initiates the request, are the foundation of cross-site request forgery (CSRF) attacks. If you visit a site of hackers or malicious content, then it could send requests to your site. The cookies associated with your browser will be accepted without any issues. Your profile or blog site needs to verify the request with counter measures because if not, attackers can easily manipulate requests to add and manipulate data.

 • Please describe how you would use the SameSite cookie mechanism to help Elgg defend against CSRF attacks. You only need to describe general ideas, and there is no need to implement them. 

Setting Same Site cookies to strict will allow cookies to only be used for a first party site. I would set the values to my SameSite cookies mechanisms to strict because it would prevent cookies from being sent by the browser to the target site in all cross site browsing contexts, including a destination from a regular url link. This would help Elgg defend against CSRF attacks because it would restrict cross site and other regular links to modify any cookies or data from Elgg.
![image](https://user-images.githubusercontent.com/82357065/196285753-7ef4e96d-6284-46d8-878f-4432f79cd90c.png)
