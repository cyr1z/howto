# howto

### [attrs howto](https://github.com/cyr1z/howto/blob/main/attrs_.ipynb)
   ``` python
   @attrs
   class Student(Person, User):
      student_id: str = attrib(default='000000')
      group_id: str = attrib(default='a0')
      course: int = attrib()
   ```


### [file system howto](https://github.com/cyr1z/howto/blob/main/File_System_Operation_Basics.ipynb) 


Show Current Directory

   ``` python
   import os
   print(f'Current Working Directory is: {os.getcwd()}')
   ```
   ```
   Current Working Directory is: /content
   ```



[Deploy Django project to Ubuntu instance]: https://github.com/cyr1z/howto/blob/main/deploy_django_to_google_cloud.md



