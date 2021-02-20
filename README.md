# howto

###[attrs howto](https://github.com/cyr1z/howto/blob/main/attrs_.ipynb)
   ``` python
   @attrs
   class Student(Person, User):
      student_id: str = attrib(default='000000')
      group_id: str = attrib(default='a0')
      course: int = attrib()
   ```
