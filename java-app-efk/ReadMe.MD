##### build the project

- ./gradlew build

##### build Docker image called java-app

- docker build -t java-app .
    
##### push image to repo 

- docker tag java-app mgrsl/java-1.0

- docker push mgrsl/java-1.0