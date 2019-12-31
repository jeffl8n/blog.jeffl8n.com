---
title: "Building Axis2 Java Classes From Wsdls Using Ant Foreach"
date: 2009-10-23T22:59:28-05:00
---

I’ve had a few requests to get an example of the build script I referenced in my post, [Ant Foreach Properties]({{< ref "/posts/ant-foreach-properties.md" >}}), so here it is.

In our ant build.xml file we have a task called buildAllWsdls which gets all of the WSDL files from the web service URL, then feeds each of them into Axis2 to convert them to Java classes.

Here are the basic steps of what this build does:

Finds each property which ends with “Service” and concatenates them into a comma delimited string (ex. ATestService, BTestService, CTestService…)

```
<propertyselector property="services.list" delimiter="," match="(w)*Service" />
```

Loop through each of those Service properties using the foreach task and get the WSDL file from the WSDL end point URL.

```
<foreach list="${services.list}" inheritall="true" target="-getSingleWsdl" delimiter="," param="prettyName" />
```

Translate the namespace to package properties file into a comma delimited list (as required by Axis 2).

```
<loadfile srcfile="${conf.dir}/NStoPkg.properties" property="ns2p.all">
     <filterchain>
         <striplinecomments>
             <comment value="//" />
         </striplinecomments>
         <prefixlines prefix="," />
         <striplinebreaks />
         <tokenfilter>
             <trim />
             <ignoreblank />
         </tokenfilter>
     </filterchain>
 </loadfile>
```

Run each WSDL in the download directory through the Axis2 Java class creation Ant task.

```
<foreach target="-createStubs" param="file" inheritall="true">
     <path>
         <fileset dir="${wsdldir}" />
     </path>
 </foreach>
 
<target name="-createStubs">
    <basename property="wsdl" file="${file}" />
    <echo message="Generating client code for service: ${wsdl}" />
    <java fork="true" classname="org.apache.axis2.wsdl.WSDL2Java" failonerror="true">
        <arg line="-ns2p ${ns2p.all} -s -u -uri ${wsdldir}/${wsdl} -o ${stubsdir} --noBuildXML" />
        <classpath refid="cp.axis" />
    </java>
</target>
```

If you don’t mind your web services being in a Java package that corresponds with the web service namespace, you can skip step 3 and leave out the -ns2p command line argument in the java ant task in step 4.

Here is the full example Ant build.xml file for doing this:

```
<project name="Axis2WSDLBuild" default="buildAllWsdls" basedir=".">
 <tstamp>
 <format property="TODAY" pattern="MM/dd/yyyy HH:mm:ss"/>
 </tstamp>
 
 <property name="ws-domain" value="http://services.test.com:8080" />
 <property name="webservices.dir" value="${app.dir}/axisStubsProject" />
 <property name="wsdldir" value="${webservices.dir}/wsdls" />
 <property name="stubsdir" value="${webservices.dir}" />
 
 <!-- patternsets -->
 <patternset id="jar.files">
 <include name="**/*.jar"/>
 </patternset>
 
 <path id="cp.axis">
 <fileset dir="${lib.dir}/axis">
 <patternset refid="jar.files"/>
 </fileset>
 </path>
 <path id="cp.antcontrib">
 <fileset dir="${lib.dir}/antcontrib">
 <patternset refid="jar.files"/>
 </fileset>
 </path>
 
 <!-- Allow usage of ant-contrib defined tasks -->
 <taskdef resource="net/sf/antcontrib/antlib.xml">
 <classpath refid="cp.antcontrib"/>
 </taskdef>
 
 <!-- =================================================================== -->
 <!-- Project initialization                                              -->
 <!-- =================================================================== -->
 <target name="init">
 <property name="version" value="build ${TODAY}"/>
 <property file="${conf.dir}/soa-addresses.properties"/>
 <filter token="version" value="${version}"/>
 <filter token="today" value="${TODAY}"/>
 </target>
 
 <!--=======================================================-->
 <!-- TARGET [buildAllWsdls]                                -->
 <!-- Target which gets and creates the web service stubs necessary for client invocation. -->
 <!--=======================================================-->
 <target name="buildAllWsdls" depends="init">
 <antcall target="wsdlGet" />
 <antcall target="buildWsdls" />
 </target>
 
 <!--=======================================================-->
 <!-- TARGET [buildWsdls]                                -->
 <!-- Target which creates the web service stubs necessary for client invocation. -->
 <!--=======================================================-->
 <target name="buildWsdls">
 <!-- create the download directory -->
 <mkdir dir="${stubsdir}" />
 <!--
 Translate the namespace to package properties file (as typically used in
 an Axis 1 implementation) into a comma delimited list (as required by
 Axis 2).
 -->
 <loadfile srcfile="${conf.dir}/NStoPkg.properties" property="ns2p.all">
 <filterchain>
 <striplinecomments>
 <comment value="//" />
 </striplinecomments>
 <prefixlines prefix="," />
 <striplinebreaks />
 <tokenfilter>
 <trim />
 <ignoreblank />
 </tokenfilter>
 </filterchain>
 </loadfile>
 <!-- run client generation for each WSDL in the download directory -->
 <foreach target="-createStubs" param="file" inheritall="true">
 <path>
 <fileset dir="${wsdldir}" />
 </path>
 </foreach>
 </target>
 
 <!--=======================================================-->
 <!-- TARGET [wsdlGet]                                -->
 <!--  Target retrieving all the WSDL files necessary for generating client code. -->
 <!--=======================================================-->
 <target name="wsdlGet">
 <!-- create the download directory -->
 <mkdir dir="${wsdldir}" />
 
 <!-- find the properties related to WSDL locations -->
 <propertyselector property="services.list" delimiter="," match="(w)*Service" />
 
 <!-- for each property found, retrieve the WSDL -->
 <foreach list="${services.list}" inheritall="true" target="-getSingleWsdl" delimiter="," param="prettyName" />
 
 </target>
 
 <!--=======================================================-->
 <!-- TARGET [-getSingleWsdl]                                -->
 <!-- Hidden Target to retrieve a single WSDL. -->
 <!--=======================================================-->
 <target name="-getSingleWsdl">
 <!-- get the property value -->
 <propertycopy property="fullPath" from="${prettyName}" />
 <!-- download the WSDL file -->
 <get src="${ws-domain}/${fullPath}?wsdl" dest="${wsdldir}/${prettyName}.wsdl" verbose="true" usetimestamp="true" />
 </target>
 
 <!--=======================================================-->
 <!-- TARGET [-createStubs]                                -->
 <!-- Hidden target to run WSDL2Java on an input file. -->
 <!--=======================================================-->
 <target name="-createStubs">
 <!-- strip the full path from the file -->
 <basename property="wsdl" file="${file}" />
 <echo message="Generating client code for service: ${wsdl}" />
 <java fork="true" classname="org.apache.axis2.wsdl.WSDL2Java" failonerror="true">
 <arg line="-ns2p ${ns2p.all} -s -u -uri ${wsdldir}/${wsdl} -o ${stubsdir} --noBuildXML" />
 <classpath refid="cp.axis" />
 </java>
 </target>
 
</project>
```

the soa-addresses.properties file, which contains the different web service URLs:

```
ATestService=services/ATestService
BTestService=services/BTestService
CTestService=services/CTestService
DTestService=services/DTestService
ETestService=services/ETestService
```

and the NStoPkg.properties file which converts the default web service namespace based off the WSDL URL to a specified java package:

```
http://typens.test.com/common=com.test.axis.common.types
http://servicens.test.com/common/ATestService=com.test.axis.common.services.atestservice
http://servicens.test.com/common/BTestService=com.test.axis.common.services.btestservice
http://servicens.test.com/common/CTestService=com.test.axis.common.services.ctestservice
http://servicens.test.com/common/DTestService=com.test.axis.common.services.dtestservice
http://servicens.test.com/common/ETestService=com.test.axis.common.services.etestservice
```