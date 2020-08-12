## CS411 Final Project Final Demo

This is a demo video for our final project Ultimate Myillini. This project aims to create a versatile mobile app to smooth U of I students' academic experience by integrating functionalities, including course scheduling, daily calendar management, course data visualization, course difficulty metrics, and instructor evaluation using data gathered by fetching official APIs and crawling third-party platforms. I hope it would be very useful for students. Also, for the project, we developed a cross-platform mobile app with react native. You can see both android and ios satisfy our project. 

So let's get started. First, here is our welcome page, you need to log in with your account to post useful information to our database system. If you forget your password, you can enter your email address, and we will send a password reset email to you. In our register screen, we use yup and formik to do input checking and form display. 

After successfully logging in, your homepage becomes with your incoming schedule and your weekly calendar. Because you are newly registered, you need to enroll in courses. Here is a popular screen. You can see some popular courses in different navigation tabs. All of our course records are fetched from Course Explorer's official API. If you are interested in students' comments on this course, you can click on the detail icon. The comments are from the Rate My Professor website that is achieved by web crawlers.

We also developed a fuzzy search engine for students to search for courses and find their favorites. Students can search for courses based on any keys. If I am going to look for all courses from the CS department, you can just enter 'CS' in the subject. Suppose you like CS173, we can add it to your favorites. If I would like to search for a specific course, like CS411, I can add course id in this field. If you click on this item, you can see the diagram for GPA distribution and average ratings of related professors. For GPA trending visualization, we used Professor Wade's highly organized GPA dataset and victory.js library to produce it. We can also see the students' comments of professors after clicking on it. Now we close the page and also add it to favorites. Here we can also search all courses by their names. 

Here are your favorites, you can enroll in these selected courses. And then, you can go to your Profile page. You can see your estimated workload in this part. It's evaluated based on past GPA information, student review, and credit hours, etc. obtained through the course schedule crawler and web scraping. Your current workload is... It's easy/medium/hard/very hard. You can also see the contribution of every course you enrolled in. Suppose you would like to cancel the course. Just click it, then it can be successfully canceled. 

After enrollments, you can see your current weekly calendar here. You can also add, delete, modify remark for them. Like... 

The countdown screen will display your incoming events. We get the nearest incoming events by comparing them with the CDT time zone. It can also navigate you to google map here. 

That's the end of our demo. We have published our app on expo platform. If you are interested, welcome to visit https://expo.io/@chenlei/expo-firebase.



```python
if workload < 5:
  return "easy"
elif 5 <= workload < 7.5:
  return "medium"
elif 7.5<= workload < 10:
  return "hard"
else return "crazy"
```







