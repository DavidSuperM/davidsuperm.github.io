# java集合排序
### 方法汇总
```
java.util.Arrays.sort(...)
java.util.Collections.sort(...)
```


排序
```
list.sort(Comparator.comparing(Student::getAge));
list.sort(Comparator.comparing(Student::getAge).thenComparing(Comparator.comparing(Student::getScore)));
list.sort(Comparator.comparing(Student::getAge).reversed());
```

### 举例
```
 public static void main(String[] args) throws Exception{
        String[] s = {"1","3","2"};
        int[] a = {1,3,2};
        Arrays.sort(s);
        Arrays.sort(a);
        System.out.println(Arrays.toString(s));
        System.out.println(Arrays.toString(a));

        Student student1 = new Student("a", 18, 90);
        Student student2 = new Student("b", 17, 91);
        Student student3 = new Student("c", 19, 89);
        Student[] students = new Student[]{student1, student2, student3};
        List<Student> list = new ArrayList<>();
        list.add(student1);
        list.add(student2);
        list.add(student3);
        Arrays.sort(students, new Comparator<Student>() {
            @Override
            public int compare(Student o1, Student o2) {
                if(o1.getAge()>o2.getAge())return 1;
                else if(o1.getAge()==o2.getAge())return 0;
                else return -1;
            }
        });
//        list.sort(Comparator.comparingInt(Student::getScore));
// 升序
        Collections.sort(list, new Comparator<Student>() {
            @Override
            public int compare(Student o1, Student o2) {
                if(o1.getScore()>o2.getScore())return 1;
                else if(o1.getScore()==o2.getScore())return 0;
                else return -1;
            }
        });
        System.out.println(Arrays.toString(students));
        System.out.println(list);
    }

    @Data
    @ToString
    @AllArgsConstructor
    @NoArgsConstructor
    @Builder
    public static class Student {
        String name;
        int age;
        int score;
    }
```
