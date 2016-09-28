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

Note, any classes use in the rules file need to have typical getters/setters.

![DRL file](/images/drools-rule-file.png)

## Other rules engines

- [nools](http://c2fo.io/nools/), for Javascript




