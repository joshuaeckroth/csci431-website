---
title: Drools rules engine
layout: note
---

# Drools rules engine

## Main method

~~~ java
KieContainer kc = KieServices.Factory.get().getKieClasspathContainer();

// load the DRL file; needs to be configured in kmodule.xml (see below)
KieSession ksession = kc.newKieSession("StudentAdvisorKS");

// insert some initial facts; perhaps you ask the user for these
Student johndoe = new Student("John", "Doe");
ksession.insert(johndoe);
Course csci221 = new Course("Software Development I", "CSCI221", 4);
Course csci142 = new Course("XYZ", "CSCI142", 4);
ksession.insert(csci221);
ksession.insert(csci142);
CourseAttempt ca = new CourseAttempt(johndoe, csci142, new Semester("Fall", 2015), 3.5);
ksession.insert(ca);

// fire all rules!
ksession.fireAllRules();
~~~

## Metadata file

Required path: `resources/META-INF/kmodule.xml`

~~~ xml
<?xml version="1.0" encoding="UTF-8"?>
<kmodule xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns="http://www.drools.org/xsd/kmodule">

    <!-- change appropriately; "KB" and "KS" seem to be required? -->
    <kbase name="StudentAdvisorKB" packages="cc.artifice.csci431.studentadvisor">
        <ksession name="StudentAdvisorKS"/>
    </kbase>
</kmodule>
~~~

## DRL (Rules) files

Example path: `resources/cc/artifice/csci431/studentadvisor/StudentAdvisor.drl`

Note, any classes used in the rules file need to have typical getters/setters.

![DRL file](/images/drools-rule-file.png)

## Queries

Besides rules, you can define queries to collect objects that satisfy certain conditions. Then, in your main Java code, you can (for example) get these objects into a list and do something with them (like print them).

DRL file:

~~~ java
query "people over the age of 30" 
    person : Person( age > 30 )
end
~~~

Java code:

~~~ java
System.out.println("These people are are over 30:");

QueryResults results = ksession.getQueryResults("people over the age of 30"); // parameter should match query name
for(QueryResultsRow row : results) {
    Person person = (Person)row.get("person");
    System.out.println(person.getName() + "\n");
}
~~~

## Other rules engines

- [nools](http://c2fo.io/nools/), for Javascript




