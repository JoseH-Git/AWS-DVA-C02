# In this hands-on lab, you will create a fully working serverless reminder application using S3, Lambda, API Gateway, Step Functions, Simple Email Service, and Simple Notification Service.

By the end of the lab, you will feel more comfortable architecting and implementing serverless solutions within AWS.

Open the lab's GitHub repository in a second tab. All of the code needed to complete this lab is available there.

## Objective 1: Verify an Email Address in Simple Email Service (SES)

1. Log in to the AWS Management Console using the credentials provided on the lab instructions page. Be sure to use an incognito or private browser window to ensure you’re using the lab account rather than your own.

   > **Note:** Make sure you're in the N. Virginia (us-east-1) region throughout the lab.

2. Open Simple Email Service in a new tab by typing Simple Email Service into the search bar at the top of the screen. Right-click Amazon Simple Email Service from the drop-down menu, and select Open Link in New Tab.

3. In the Amazon Simple Email Service (SES) console, in the left-hand menu under Configuration choose Identities.

4. Click Create identity.

5. Choose Email address, enter your personal email address (or an email address you created specifically for this hands-on lab). Click Create identity. (Note outlook accounts do not seem to work well with this lab but gmail does).

6. In a new browser tab or email client, navigate to your email, open the Amazon SES verification email, and click the provided link.

   > **Note:** Check your spam mail if you do not see it in your inbox.

7. You should see a Congratulations! page that confirms you successfully verified your email address.

8. Return to the AWS SES console tab.

9. You should now see Verified under Identity status. (Note: You may need to click the Refresh button.)

---

## Objective 2: Create the Email Lambda Function

1. In the GitHub repo GitHub Repo, click Code > Download .ZIP.

2. Unzip the file, and using a text editor such as VSCode open the file, `email_reminder.py`.

3. Return to your first tab, and in the AWS Management Console, navigate to Lambda by typing Lambda in the search bar at the top of the screen. Select Lambda from the drop-down menu.

4. In the AWS Lambda console, click Create a function.

5. Leave the Author from scratch option selected, and then set the following values:
   - **Function name:** email
   - **Runtime:** Python 3.12

6. Expand Change default execution role by clicking the arrow next to it.

7. Below Execution role, select Use an existing role.

8. Once you select that, click into the empty drop-down menu that appears below Existing role.

9. Select LambdaRuntimeRole from the drop-down menu.

10. Click Create function at the bottom of the page.

11. Close any pop-ups, scroll down to Code source and replace the provided code in `lambda_function.py` with the code from `email_reminder.py`.

12. Within the lambda_function code, replace the `YOUR_SES_VERIFIED_EMAIL` placeholder with your email, making sure to leave the single quotes around it.

13. On the left, a little bit lower down, click Deploy.

14. Your changes have now been deployed.

15. Scroll up on the same page to the Function Overview section, and copy the Function ARN by clicking the copy icon next to it, and paste it into a text file for later use in the lab.

---

## Objective 3: Create a Step Function State Machine

1. Open the AWS Step Functions console in a new tab by typing Step Functions into the search bar at the top of the screen. Right-click Step Functions from the drop-down menu, and select Open Link in New Tab.

2. In the AWS Step Functions console tab, click Get started.

3. In the pop-up window, click Create your own to create your own workflow from scratch.

4. In the next pop-up, enter `MyStateMachine`, and click Continue.

5. On the new page at the top, click on `{}` Code.

6. Overwrite the templated json on the left side of the window with the code from the GitHub repo in the `step-function-template.json` file.

7. Replace the `EMAIL_REMINDER_ARN` placeholder value with the Function ARN you copied earlier, being sure to leave the double quotes around the ARN.

8. Click on the Config button at the top.

9. Under Permissions, click the drop-down for the Execution role and select `RoleForStepFunction`.

10. In the upper-right, click Create.

11. On the MyStateMachine page, under Details, copy the Arn and store it locally for later use.

---

## Objective 4: Create the api_handler Lambda Function

1. Scroll up to the navigation line at the top of the page, click Lambda to return to the main Lambda console.

2. Click Create function.

3. Leave the Author from scratch option selected, and then set the following values:
   - **Function name:** api_handler
   - **Runtime:** Python 3.12

4. Expand Change default execution role by clicking the arrow next to it.

5. Below Execution role, select Use an existing role.

6. Once you select that, click into the empty drop-down menu that appears below Existing role.

7. Select LambdaRuntimeRole from the drop-down menu.

8. Click Create function at the bottom of the page.

9. Close any pop-ups, scroll down to Code source and replace the provided code in `lambda_function.py` with the code from `api_handler.py` (from the GitHub repo).

10. Within the code, replace the `STEP_FUNCTION_ARN` placeholder with the MyStateMachine Arn you stored earlier, making sure to leave the single quotes around it.

11. Click Deploy. (Keep this browser tab open.)

---

## Objective 5: Create the API Gateway

1. Open the Amazon API Gateway console in a new tab by typing API Gateway into the search bar at the top of the screen. Right-click API Gateway from the drop-down menu, and select Open Link in New Tab.

2. In your Amazon API Gateway console tab, scroll down a bit and click Create Rest API.

3. Under API Details, select New API.

4. Then configure the following values:
   - **API name:** reminders
   - Leave Description blank.
   - **Endpoint Type:** Regional

5. Click Create API at the bottom of the screen.

6. You should now be in your API Gateway, and you will see that you don't have any methods defined for the resource.

7. You should now see the Resources page. Click Create resource.

8. Under Resource Name, enter `reminders`.

9. Select the checkbox next to CORS.

10. Click Create resource at the bottom of the page.

11. You should be back at the Resources page again.

12. Click Create method.

13. In the drop-down that appears under Method type, select POST and set the following values:
   - **Integration type:** Lambda function
   - Enable Lambda proxy integration by clicking the toggle.
   - **Lambda function:** us-east-1, and in the box to the right select the one that ends in `api_handler`.

14. Click Create method at the bottom of the page.

15. Back at the Resources page, click Deploy API at the top.

16. In the Deploy API pop-up window, set the following values:
   - **Deployment stage:** *New Stage*
   - **Stage name:** prod

17. Click Deploy.

   > **Note:** You may ignore any Web Application Firewall (WAF) permissions warning messages if received after deployment.

18. Copy the Invoke URL and store it locally for use in the next objective.

---

## Objective 6: Create and Test the Static S3 Website

1. In the GitHub files you downloaded earlier, in the `static_website` folder, open `formlogic.js` locally in a text editor.

2. Replace the `UPDATETOYOURINVOKEURLENDPOINT` placeholder with the Invoke URL you copied earlier.

3. Keep the single quotes, and keep the `/reminders` suffix.

4. Save the file.

5. Back in the AWS Console, open the Amazon S3 console in a new tab by typing S3 into the search bar at the top of the screen. Right-click S3 from the drop-down menu, and select Open Link in New Tab.

6. In the Amazon S3 console, click Create bucket.

7. On the Create bucket page, under Bucket Type leave General Purpose selected, and then for Bucket name, enter a globally unique bucket name.

8. Under Object Ownership, select ACLs enabled, and ensure Bucket owner preferred is selected.

9. Uncheck Block all public access.

10. Under Block all public access, select I acknowledge that the current settings might result in this bucket and the objects within becoming public.

11. Leave the other defaults, scroll down, and click Create bucket.

12. Click the new bucket's link (under the Name column) to open it.

13. On the bucket page, click Upload.

14. On the Upload page, click Add files and select all of your local files from the `static_website` folder.

15. You will now see the following files under Files and folders:
    - cat.png
    - error.html
    - formlogic.js
    - IMG_0991.jpg
    - index.html
    - main.css

16. Once all of the files have been added, scroll down, and click Permissions to expand the access options.

17. Under Predefined ACLs, select Grant public-read access.

18. Under Grant public-read access, select I understand the risk of granting public-read access to the specified objects.

19. Click Upload at the bottom of the page.

20. Once you see the uploads have succeeded, click Close in the top right-hand corner of the page.

21. Click on the Properties tab under the bucket name.

22. Scroll down to Static website hosting, and click Edit.

23. On the Edit static website hosting page, under Static website hosting, select Enable.

24. Set the following values:
    - **Index document:** index.html
    - **Error document:** error.html

25. Click Save changes at the bottom of the page.

26. Scroll down again to Static website hosting, and click the URL below Bucket website endpoint to access the webpage.

27. Continue to the site, ignoring any security warnings as it's okay for this lab.

28. Once you are on the static website, to test the service's functionality, set the following values:
    - **Seconds to wait:** 1
    - **Message:** Hello!
    - **Email address required for email reminders:** Your email address that you verified with Simple Email Service earlier.

29. At the bottom, click email.

30. You will see *Looks ok. But check the result below!* above the Required sections, and:

```json
{"Status":"Success"}
```

at the bottom of the page.

31. Navigate to your AWS Step Functions browser tab.

32. In the Executions section, click on the Refresh icon.

33. You will see at least one execution displayed.

34. Click on one of the executions (at the bottom, under the Name column).

35. Scroll down to Graph view to view the event's visual workflow.

36. In a new browser tab or email client, navigate to your email. You should now see a reminder email from the service.

   > **Note:** Check your spam folder if you do not see it in your inbox. And, due to your email settings especially for corporate emails, you may get an error from your mailer daemon as it was blocked.

---

# Congratulations—you've completed this hands-on lab!