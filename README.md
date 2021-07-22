<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.4.0</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.hcentive.exchange</groupId>
	<artifactId>magi-rules-proxy</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>magi-rules-proxy</name>
	<description>Demo project for Spring Boot</description>

	<properties>
		<java.version>1.8</java.version>
		<jboss.brms.version>7.8.0.redhat-00002</jboss.brms.version>
	</properties>
	
	<repositories>
		<repository>
			<id>jboss-maven-repository</id>
			<name>JBoss Maven Repository</name>
			<url>https://maven.repository.redhat.com/ga/</url>
			<layout>default</layout>
			<releases>
				<enabled>true</enabled>
				<updatePolicy>never</updatePolicy>
			</releases>
			<snapshots>
				<enabled>false</enabled>
				<updatePolicy>never</updatePolicy>
			</snapshots>
		</repository>
	</repositories>
	
	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>com.redhat.ba</groupId>
				<artifactId>ba-platform-bom</artifactId>
				<type>pom</type>
				<scope>import</scope>
				<version>${jboss.brms.version}</version>
			</dependency>
		</dependencies>
	</dependencyManagement>

	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
		
		<!-- Project dependencies -->
		<dependency>
			<groupId>org.kie</groupId>
			<artifactId>kie-ci</artifactId>
		</dependency>
		<dependency>
			<groupId>org.kie</groupId>
			<artifactId>kie-dmn-core</artifactId>
		</dependency>
		<dependency>
			<groupId>org.kie.server</groupId>
			<artifactId>kie-server-client</artifactId>
		</dependency>
		<dependency>
			<groupId>com.ma-hix-rules</groupId>
			<artifactId>MA-HIX-MAGI</artifactId>
			<version>1.0.0-SNAPSHOT</version>
		</dependency>
		<dependency>
			<groupId>junit</groupId>
			<artifactId>junit</artifactId>
			<scope>test</scope>
		</dependency>
		
	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>

</project>


package com.dmn.test.embedded;

import static org.assertj.core.api.Assertions.assertThat;

import java.math.BigDecimal;
import java.util.Map;
import java.util.UUID;

import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.junit.runner.RunWith;
import org.kie.api.KieServices;
import org.kie.api.runtime.KieContainer;
import org.kie.dmn.api.core.DMNContext;
import org.kie.dmn.api.core.DMNDecisionResult;
import org.kie.dmn.api.core.DMNModel;
import org.kie.dmn.api.core.DMNResult;
import org.kie.dmn.api.core.DMNRuntime;
import org.kie.dmn.core.compiler.RuntimeTypeCheckOption;
import org.kie.dmn.core.impl.DMNRuntimeImpl;
import org.kie.dmn.core.util.KieHelper;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

@RunWith(SpringJUnit4ClassRunner.class)
public class EmbeddedTest {
	
	private DMNRuntime dmnRuntime;
	
	@BeforeEach
	public void getReady() {
		final KieServices ks = KieServices.Factory.get();
        final KieContainer kieContainer = KieHelper.getKieContainer(
                ks.newReleaseId("org.kie", "dmn-test-"+UUID.randomUUID(), "1.0"),
                ks.getResources().newClassPathResource("\\com\\dmn\\test\\embedded\\demo.dmn", this.getClass()));
        dmnRuntime = typeSafeGetKieRuntime(kieContainer);
	}
	
	public static DMNRuntime typeSafeGetKieRuntime(final KieContainer kieContainer) {
        DMNRuntime dmnRuntime = kieContainer.newKieSession().getKieRuntime(DMNRuntime.class);
        ((DMNRuntimeImpl) dmnRuntime).setOption(new RuntimeTypeCheckOption(true));
        return dmnRuntime;
    }
	
	@Test
	public void test_001() throws Exception {
		
		String namespace = "https://kiegroup.org/dmn/_80B2335A-77E6-467B-85C4-C3D1305B27C4";
		String modelName = "demoRule";

		DMNModel dmnModel = dmnRuntime.getModel(namespace, modelName);
		DMNContext dmnContext = dmnRuntime.newContext();
		
		//TODO: Change the input as per your requirement
		dmnContext.set("demoFact_1", true);
		dmnContext.set("demoFact_2", true);
		
		DMNResult dmnResult = dmnRuntime.evaluateAll(dmnModel, dmnContext);
		for (DMNDecisionResult dr : dmnResult.getDecisionResults()) {
			Map response = (Map) dr.getResult();
			assertThat((String) response.get("outcome_1")).isEqualTo("sampleOutcome_1");
	  }
	}
	
	@Test
	public void test_002() throws Exception {
		
		String namespace = "https://kiegroup.org/dmn/_80B2335A-77E6-467B-85C4-C3D1305B27C4";
		String modelName = "demoRule";

		DMNModel dmnModel = dmnRuntime.getModel(namespace, modelName);
		DMNContext dmnContext = dmnRuntime.newContext();
		
		//TODO: Change the input as per your requirement
		dmnContext.set("demoFact_1", true);
		dmnContext.set("demoFact_2", false);
		
		DMNResult dmnResult = dmnRuntime.evaluateAll(dmnModel, dmnContext);
		for (DMNDecisionResult dr : dmnResult.getDecisionResults()) {
			Map response = (Map) dr.getResult();
			assertThat((String) response.get("outcome_1")).isEqualTo("sampleOutcome_2");
	  }
	}
}

demo.dmn
<?xml version='1.0' encoding='UTF-8'?>
<dmn:definitions xmlns:dmn="http://www.omg.org/spec/DMN/20180521/MODEL/" xmlns="https://kiegroup.org/dmn/_80B2335A-77E6-467B-85C4-C3D1305B27C4" xmlns:di="http://www.omg.org/spec/DMN/20180521/DI/" xmlns:kie="http://www.drools.org/kie/dmn/1.2" xmlns:feel="http://www.omg.org/spec/DMN/20180521/FEEL/" xmlns:dmndi="http://www.omg.org/spec/DMN/20180521/DMNDI/" xmlns:dc="http://www.omg.org/spec/DMN/20180521/DC/" id="_25E4B3E9-CFE7-4F49-9A61-1A46B8ADC7F2" name="test" expressionLanguage="http://www.omg.org/spec/DMN/20180521/FEEL/" typeLanguage="http://www.omg.org/spec/DMN/20180521/FEEL/" namespace="https://kiegroup.org/dmn/_80B2335A-77E6-467B-85C4-C3D1305B27C4">
  <dmn:extensionElements/>
  <dmn:inputData id="_58ABFA0B-8204-4588-9A1C-A20C39B826F0" name="demoFact_2">
    <dmn:extensionElements/>
    <dmn:variable id="_9D090A82-0CCB-41BD-B563-FC274F6C4A18" name="demoFact_2" typeRef="boolean"/>
  </dmn:inputData>
  <dmn:inputData id="_017A40B3-D270-4379-AE0F-AEC0144248D3" name="demoFact_1">
    <dmn:extensionElements/>
    <dmn:variable id="_FD7C8A6E-2BB3-4681-8DF1-9C0DC53A2F7C" name="demoFact_1" typeRef="boolean"/>
  </dmn:inputData>
  <dmn:decision id="_95FD48BD-2DBC-42C4-985C-61C097DB9926" name="outcome_1">
    <dmn:extensionElements/>
    <dmn:variable id="_25E5A41E-5A53-4E14-8DD0-F8200B4DD702" name="outcome_1" typeRef="string"/>
    <dmn:informationRequirement id="_03CF5E4D-F3A5-451E-A9B8-80E856142602">
      <dmn:requiredInput href="#_58ABFA0B-8204-4588-9A1C-A20C39B826F0"/>
    </dmn:informationRequirement>
    <dmn:informationRequirement id="_49F3082A-0E89-4D5B-865F-556C0D0DDB61">
      <dmn:requiredInput href="#_017A40B3-D270-4379-AE0F-AEC0144248D3"/>
    </dmn:informationRequirement>
    <dmn:decisionTable id="_AFB3459E-DB5F-4FF0-B6DE-93628831B9AD" hitPolicy="ANY" preferredOrientation="Rule-as-Row">
      <dmn:input id="_4BDC26E2-AF06-47FF-9DE3-01610BCE3498">
        <dmn:inputExpression id="_1C021B60-0CC7-4D45-92F3-3839CD642C4E" typeRef="boolean">
          <dmn:text>demoFact_1</dmn:text>
        </dmn:inputExpression>
      </dmn:input>
      <dmn:input id="_8FDF3D5F-8773-41D7-995C-96649E4B8899">
        <dmn:inputExpression id="_60D8FD1C-3F77-437B-93B4-493A55A4BF43" typeRef="boolean">
          <dmn:text>demoFact_2</dmn:text>
        </dmn:inputExpression>
      </dmn:input>
      <dmn:output id="_CFEAF6A6-B227-4466-B00C-E59FB60D0590"/>
      <dmn:annotation name="annotation-1"/>
      <dmn:rule id="_D33E1135-4A24-4EA4-999D-BE4B3FA19049">
        <dmn:inputEntry id="_1051974E-4CB4-4DB0-8E06-A0AD5E79973D">
          <dmn:text>true</dmn:text>
        </dmn:inputEntry>
        <dmn:inputEntry id="_B01FA26B-8708-4B3D-88E2-C0E52459A08F">
          <dmn:text>true</dmn:text>
        </dmn:inputEntry>
        <dmn:outputEntry id="_A3DFB1B3-34ED-4F5A-A404-D4BF9BA30CB4">
          <dmn:text>"sampleOutcome_1"</dmn:text>
        </dmn:outputEntry>
        <dmn:annotationEntry>
          <dmn:text></dmn:text>
        </dmn:annotationEntry>
      </dmn:rule>
      <dmn:rule id="_E4852AAB-A8FA-4E5A-8010-EBB506E0F3B4">
        <dmn:inputEntry id="_3A5EA5C1-A044-4D2D-88BE-E6CCBA7301D6">
          <dmn:text>true</dmn:text>
        </dmn:inputEntry>
        <dmn:inputEntry id="_CAD0F5C0-B1E2-435B-A96D-6B990CE4043B">
          <dmn:text>false</dmn:text>
        </dmn:inputEntry>
        <dmn:outputEntry id="_26113E74-44C3-4770-9261-2773B14FC8BE">
          <dmn:text>"sampleOutcome_2"</dmn:text>
        </dmn:outputEntry>
        <dmn:annotationEntry>
          <dmn:text></dmn:text>
        </dmn:annotationEntry>
      </dmn:rule>
    </dmn:decisionTable>
  </dmn:decision>
  <dmndi:DMNDI>
    <dmndi:DMNDiagram>
      <di:extension>
        <kie:ComponentsWidthsExtension>
          <kie:ComponentWidths dmnElementRef="_AFB3459E-DB5F-4FF0-B6DE-93628831B9AD">
            <kie:width>50.0</kie:width>
            <kie:width>100.0</kie:width>
            <kie:width>100.0</kie:width>
            <kie:width>258.0</kie:width>
            <kie:width>100.0</kie:width>
          </kie:ComponentWidths>
        </kie:ComponentsWidthsExtension>
      </di:extension>
      <dmndi:DMNShape id="dmnshape-_017A40B3-D270-4379-AE0F-AEC0144248D3" dmnElementRef="_017A40B3-D270-4379-AE0F-AEC0144248D3" isCollapsed="false">
        <dmndi:DMNStyle>
          <dmndi:FillColor red="255" green="255" blue="255"/>
          <dmndi:StrokeColor red="0" green="0" blue="0"/>
          <dmndi:FontColor red="0" green="0" blue="0"/>
        </dmndi:DMNStyle>
        <dc:Bounds x="485" y="250" width="100" height="50"/>
        <dmndi:DMNLabel/>
      </dmndi:DMNShape>
      <dmndi:DMNShape id="dmnshape-_95FD48BD-2DBC-42C4-985C-61C097DB9926" dmnElementRef="_95FD48BD-2DBC-42C4-985C-61C097DB9926" isCollapsed="false">
        <dmndi:DMNStyle>
          <dmndi:FillColor red="255" green="255" blue="255"/>
          <dmndi:StrokeColor red="0" green="0" blue="0"/>
          <dmndi:FontColor red="0" green="0" blue="0"/>
        </dmndi:DMNStyle>
        <dc:Bounds x="566" y="124" width="100" height="50"/>
        <dmndi:DMNLabel/>
      </dmndi:DMNShape>
      <dmndi:DMNShape id="dmnshape-_58ABFA0B-8204-4588-9A1C-A20C39B826F0" dmnElementRef="_58ABFA0B-8204-4588-9A1C-A20C39B826F0" isCollapsed="false">
        <dmndi:DMNStyle>
          <dmndi:FillColor red="255" green="255" blue="255"/>
          <dmndi:StrokeColor red="0" green="0" blue="0"/>
          <dmndi:FontColor red="0" green="0" blue="0"/>
        </dmndi:DMNStyle>
        <dc:Bounds x="651" y="245" width="100" height="50"/>
        <dmndi:DMNLabel/>
      </dmndi:DMNShape>
      <dmndi:DMNEdge id="dmnedge-_03CF5E4D-F3A5-451E-A9B8-80E856142602" dmnElementRef="_03CF5E4D-F3A5-451E-A9B8-80E856142602">
        <di:waypoint x="701" y="270"/>
        <di:waypoint x="616" y="149"/>
      </dmndi:DMNEdge>
      <dmndi:DMNEdge id="dmnedge-_49F3082A-0E89-4D5B-865F-556C0D0DDB61" dmnElementRef="_49F3082A-0E89-4D5B-865F-556C0D0DDB61">
        <di:waypoint x="535" y="275"/>
        <di:waypoint x="616" y="149"/>
      </dmndi:DMNEdge>
    </dmndi:DMNDiagram>
  </dmndi:DMNDI>
</dmn:definitions>

