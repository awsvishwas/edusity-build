# edusity-build

**Note:** 

> Code of the application is available on a different repository. [Visit the Edusity Frontend Website Github Repository here](https://github.com/awsvishwas/edusity-frontend-website)
> This repository is created only to demonstrate how to deploy a React application on AWS ECS using Github Actions.
> This repository only contains build package for edusity frontend website.

<br><br>
** Steps to Deploy the Application: **<br><br>

1. Create a Workflow with Github Actions for Deployment on ECS. This create a file with aws.yml extension. <br>
   
![Screenshot 2024-08-06 180911](https://github.com/user-attachments/assets/22cb8da0-0a3e-4f56-9a7d-7605c989fd05)
![Screenshot 2024-08-06 181117](https://github.com/user-attachments/assets/8623953b-ae63-47c7-b11b-f0758eba7487)

<br><br>

2. After this, create an ECR Registry, ECS Cluster with AWS Management Console.<br><br>
   
![Screenshot 2024-08-06 181954](https://github.com/user-attachments/assets/e0bcb86a-78df-42fc-8b6b-b86936139445)

<br><br>
![Screenshot 2024-08-06 183019](https://github.com/user-attachments/assets/466e5065-7ac7-4fa0-912f-9b76f0dbdb7d)

3. To allow Github to write docker images to ECR and have access to ECS tasks later on, lets define a Github Identity provider<br><br>

![Screenshot 2024-08-06 193704](https://github.com/user-attachments/assets/ae6c75fb-0764-434b-b0c2-e8d40adfdbf2)

<br><br>

4. Associate a role with ECS_FullAccess, ElasticContainerRegistryPublic_FullAccess, EC2ContainerRegistryFullAcess policies to this identity provider.<br><br>
   
![Screenshot 2024-08-06 194520](https://github.com/user-attachments/assets/2421fe5f-a72d-4c29-8546-3564df5d8700)

<br><br>
5. Copy the ARN of the IAM role created, to be used in workflow file as the role to be assumed for the build job.<br><br>
![Screenshot 2024-08-06 194536](https://github.com/user-attachments/assets/5a5a2382-51af-4236-83e7-eb3664cb018d)
<br>
![Screenshot 2024-08-06 195736](https://github.com/user-attachments/assets/a80f65fe-59b5-4fd7-b153-0b5f94d0eb88)

<br><br>
6. Create a Dockerfile to build a docker image with the build steps below:<br>
<br>
```bash
#Stage1: Build
FROM node:18-alpine as build
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY src/ src/
RUN npm run build

#Stage2: Serve
FROM nginx:alpine
COPY --from=build /app/dist /usr/share/nginx/html

```
<br>
7. Commit the changes, As a result a docker image is pushed to the ECR registry, same can be verified in the console output during workflow execution.<br><br>

![Screenshot 2024-08-06 215913](https://github.com/user-attachments/assets/d3f58724-75f6-4f99-8ff7-c2713b55008a)
<br>
![Screenshot 2024-08-06 215937](https://github.com/user-attachments/assets/a601b09c-63d6-441b-8e99-314decaf977a)
<br><br>
8. This is how aws.yml file would look after build job execution.<br><br>
<br>   
![Screenshot 2024-08-06 220141](https://github.com/user-attachments/assets/452a6c17-78be-496e-a521-c66ca23909ca)
<br><br>
9. Now, lets create a task definition for AWS ECS containers.<br><br>
<br>    
![Screenshot 2024-08-06 220223](https://github.com/user-attachments/assets/8a2173cd-e3f7-4135-a3ba-3134e6ada0dc)
<br>
![Screenshot 2024-08-06 220414](https://github.com/user-attachments/assets/0fa56894-c330-4875-b92f-18baf29e5583)
<br>
![Screenshot 2024-08-06 220548](https://github.com/user-attachments/assets/3f66e237-6246-460b-9b98-abcf5f15e881)
<br><br>

10. Manually, deploy the application by creating a service.<br><br>
<br>
 
![Screenshot 2024-08-06 220621](https://github.com/user-attachments/assets/5b1fae79-2444-4459-88b9-8fe6620d870c)
<br><br>
11. Service will create a task, In Task, go to Networking => Copy the public ip of the container, paste on the browser. The application should be accessible.<br><br>
<br>    
![Screenshot 2024-08-07 145418](https://github.com/user-attachments/assets/4f13c230-1ee2-4000-a4e7-a0fdf71622b0)
<br><br>
12. Update the values in aws.yml file for ECS_CLUSTER, ECS_SERVICE, ECS_CONTAINER etc.<br>

![Screenshot 2024-08-07 150901](https://github.com/user-attachments/assets/d900fd8a-12f5-4f72-ac6f-029cd6ecf905)
<br><br>

13. Now, to automate this, Copy the Json version of the ECS Task definition. Create a new directory in your github repo with name .aws.
    Inside the directory, create a new file with extension tesk-definition.json and paste the JSON content copied for AWS ECS task definition.

    Commit and Push the changes. It will trigger the workflow execution. <br><br>
    
![Screenshot 2024-08-07 150345](https://github.com/user-attachments/assets/21bbf41d-97b8-4069-b227-46b6fceeced8)
<br>
![Screenshot 2024-08-07 154517](https://github.com/user-attachments/assets/5ebabc49-0755-4c8d-a774-f20add2af901)
<br>
![Screenshot 2024-08-07 154526](https://github.com/user-attachments/assets/c6087023-8245-44fd-aa70-a4269617d030)
<br><br>

14. After the Successful deployment, a new task will created in the same ECS Service, access it via its public ip.<br><br>
![Screenshot 2024-08-07 154429](https://github.com/user-attachments/assets/84010474-57aa-4924-8f91-7cdfdba059a0)<br>
![Screenshot 2024-08-07 154616](https://github.com/user-attachments/assets/326679fa-8724-490f-822c-cd3c7748ed79)
<br><br>
15. Now, lets add Integration test and rollback stage in our deployment.<br><br>
![Screenshot 2024-08-07 160047](https://github.com/user-attachments/assets/88330d82-639d-424e-b562-4d262d65a4c5)
<br><br>

16. Successful execution of the entire pipeline.<br><br>  
![Screenshot 2024-08-07 161020](https://github.com/user-attachments/assets/09430ce2-2def-4aea-9da1-8684ecb3ea37)
<br><br>


Thanks for visiting.<br><br>
