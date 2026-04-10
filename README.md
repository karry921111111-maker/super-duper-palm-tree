#!/usr/bin/env python3
# -*- coding: utf-8 -*-
"""
完整的项目文件生成脚本
自动创建所有backend、frontend、ml-service和配置文件
"""

import os
import sys

# 定义所有需要创建的文件及其内容
FILES_TO_CREATE = {
    # ==================== BACKEND FILES ====================
    
    # Backend - pom.xml
    "backend/pom.xml": '''<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.chronicdisease</groupId>
    <artifactId>health-profile-saas</artifactId>
    <version>1.0.0</version>
    <packaging>jar</packaging>
    <name>Chronic Disease Health Profile SaaS</name>
    <description>SaaS platform for chronic disease health profiles</description>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.7.14</version>
        <relativePath/>
    </parent>
    <properties>
        <java.version>11</java.version>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
            <version>42.6.0</version>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt</artifactId>
            <version>0.11.5</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
        <dependency>
            <groupId>io.lettuce</groupId>
            <artifactId>lettuce-core</artifactId>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springdoc</groupId>
            <artifactId>springdoc-openapi-ui</artifactId>
            <version>1.7.0</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-lang3</artifactId>
            <version>3.12.0</version>
        </dependency>
    </dependencies>
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <excludes>
                        <exclude>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>''',

    # Backend - Dockerfile
    "backend/Dockerfile": '''FROM maven:3.9-eclipse-temurin-11 as build
WORKDIR /build
COPY pom.xml .
RUN mvn dependency:go-offline
COPY src ./src
RUN mvn clean package -DskipTests

FROM eclipse-temurin:11-jre-alpine
WORKDIR /app
COPY --from=build /build/target/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
''',

    # Backend - Main Application Class
    "backend/src/main/java/com/chronicdisease/ChronicDiseaseApplication.java": '''package com.chronicdisease;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cache.annotation.EnableCaching;
import org.springframework.scheduling.annotation.EnableScheduling;

@SpringBootApplication
@EnableCaching
@EnableScheduling
public class ChronicDiseaseApplication {

    public static void main(String[] args) {
        SpringApplication.run(ChronicDiseaseApplication.class, args);
    }

}
''',

    # Backend - TenantContext
    "backend/src/main/java/com/chronicdisease/util/TenantContext.java": '''package com.chronicdisease.util;

import java.util.UUID;

public class TenantContext {
    private static final ThreadLocal<UUID> tenantId = new ThreadLocal<>();

    public static void setTenantId(UUID id) {
        tenantId.set(id);
    }

    public static UUID getTenantId() {
        return tenantId.get();
    }

    public static void clear() {
        tenantId.remove();
    }
}
''',

    # Backend - Tenant Entity
    "backend/src/main/java/com/chronicdisease/entity/Tenant.java": '''package com.chronicdisease.entity;

import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;
import org.hibernate.annotations.GenericGenerator;

import javax.persistence.*;
import java.time.LocalDateTime;
import java.util.UUID;

@Entity
@Table(name = "tenants")
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class Tenant {

    @Id
    @GeneratedValue(generator = "uuid2")
    @GenericGenerator(name = "uuid2", strategy = "uuid2")
    @Column(columnDefinition = "uuid")
    private UUID id;

    @Column(nullable = false, unique = true)
    private String name;

    @Column(nullable = false)
    private String status;

    @Column(name = "created_at", nullable = false, updatable = false)
    private LocalDateTime createdAt;

    @Column(name = "updated_at")
    private LocalDateTime updatedAt;

    @PrePersist
    protected void onCreate() {
        createdAt = LocalDateTime.now();
        updatedAt = LocalDateTime.now();
    }

    @PreUpdate
    protected void onUpdate() {
        updatedAt = LocalDateTime.now();
    }
}
''',

    # Backend - Patient Entity
    "backend/src/main/java/com/chronicdisease/entity/Patient.java": '''package com.chronicdisease.entity;

import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;
import org.hibernate.annotations.GenericGenerator;

import javax.persistence.*;
import java.time.LocalDate;
import java.time.LocalDateTime;
import java.util.UUID;

@Entity
@Table(name = "patients")
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class Patient {

    @Id
    @GeneratedValue(generator = "uuid2")
    @GenericGenerator(name = "uuid2", strategy = "uuid2")
    @Column(columnDefinition = "uuid")
    private UUID id;

    @ManyToOne
    @JoinColumn(name = "tenant_id", nullable = false)
    private Tenant tenant;

    @Column(nullable = false)
    private String name;

    @Column(nullable = false)
    private Integer age;

    @Column(nullable = false)
    private String gender;

    @Column(name = "id_number")
    private String idNumber;

    @Column(name = "phone_number")
    private String phoneNumber;

    @Column(name = "birth_date")
    private LocalDate birthDate;

    @Column(columnDefinition = "TEXT")
    private String medicalHistory;

    @Column(name = "created_at", nullable = false, updatable = false)
    private LocalDateTime createdAt;

    @Column(name = "updated_at")
    private LocalDateTime updatedAt;

    @PrePersist
    protected void onCreate() {
        createdAt = LocalDateTime.now();
        updatedAt = LocalDateTime.now();
    }

    @PreUpdate
    protected void onUpdate() {
        updatedAt = LocalDateTime.now();
    }
}
''',

    # Backend - HealthMetric Entity
    "backend/src/main/java/com/chronicdisease/entity/HealthMetric.java": '''package com.chronicdisease.entity;

import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;
import org.hibernate.annotations.GenericGenerator;

import javax.persistence.*;
import java.time.LocalDateTime;
import java.util.UUID;

@Entity
@Table(name = "health_metrics")
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class HealthMetric {

    @Id
    @GeneratedValue(generator = "uuid2")
    @GenericGenerator(name = "uuid2", strategy = "uuid2")
    @Column(columnDefinition = "uuid")
    private UUID id;

    @ManyToOne
    @JoinColumn(name = "patient_id", nullable = false)
    private Patient patient;

    @Column(name = "metric_type", nullable = false)
    private String metricType;

    @Column(nullable = false)
    private Float value;

    @Column(nullable = false)
    private String unit;

    @Column(name = "recorded_at", nullable = false)
    private LocalDateTime recordedAt;

    @Column(name = "created_at", nullable = false, updatable = false)
    private LocalDateTime createdAt;

    @PrePersist
    protected void onCreate() {
        createdAt = LocalDateTime.now();
    }
}
''',

    # Backend - ChronicDisease Entity
    "backend/src/main/java/com/chronicdisease/entity/ChronicDisease.java": '''package com.chronicdisease.entity;

import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;
import org.hibernate.annotations.GenericGenerator;

import javax.persistence.*;
import java.time.LocalDate;
import java.time.LocalDateTime;
import java.util.UUID;

@Entity
@Table(name = "chronic_diseases")
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class ChronicDisease {

    @Id
    @GeneratedValue(generator = "uuid2")
    @GenericGenerator(name = "uuid2", strategy = "uuid2")
    @Column(columnDefinition = "uuid")
    private UUID id;

    @ManyToOne
    @JoinColumn(name = "patient_id", nullable = false)
    private Patient patient;

    @Column(name = "disease_code")
    private String diseaseCode;

    @Column(name = "disease_name", nullable = false)
    private String diseaseName;

    @Column(name = "diagnosed_at", nullable = false)
    private LocalDate diagnosedAt;

    @Column(columnDefinition = "TEXT")
    private String notes;

    @Column(name = "created_at", nullable = false, updatable = false)
    private LocalDateTime createdAt;

    @PrePersist
    protected void onCreate() {
        createdAt = LocalDateTime.now();
    }
}
''',

    # Backend - HealthProfile Entity
    "backend/src/main/java/com/chronicdisease/entity/HealthProfile.java": '''package com.chronicdisease.entity;

import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;
import org.hibernate.annotations.GenericGenerator;

import javax.persistence.*;
import java.time.LocalDateTime;
import java.util.UUID;

@Entity
@Table(name = "health_profiles")
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class HealthProfile {

    @Id
    @GeneratedValue(generator = "uuid2")
    @GenericGenerator(name = "uuid2", strategy = "uuid2")
    @Column(columnDefinition = "uuid")
    private UUID id;

    @OneToOne
    @JoinColumn(name = "patient_id", nullable = false, unique = true)
    private Patient patient;

    @Column(name = "risk_level", nullable = false)
    private String riskLevel;

    @Column(name = "risk_score", nullable = false)
    private Float riskScore;

    @Column(columnDefinition = "TEXT")
    private String identifiedDiseases;

    @Column(columnDefinition = "TEXT")
    private String recommendations;

    @Column(name = "updated_at", nullable = false)
    private LocalDateTime updatedAt;

    @PrePersist
    protected void onCreate() {
        updatedAt = LocalDateTime.now();
    }

    @PreUpdate
    protected void onUpdate() {
        updatedAt = LocalDateTime.now();
    }
}
''',

    # Backend - PatientDTO
    "backend/src/main/java/com/chronicdisease/dto/PatientDTO.java": '''package com.chronicdisease.dto;

import com.fasterxml.jackson.annotation.JsonProperty;
import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;

import javax.validation.constraints.*;
import java.time.LocalDate;
import java.util.UUID;

@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class PatientDTO {

    private UUID id;

    @NotBlank(message = "Patient name is required")
    private String name;

    @NotNull(message = "Age is required")
    @Min(0)
    @Max(150)
    private Integer age;

    @NotBlank(message = "Gender is required")
    @Pattern(regexp = "[MFO]", message = "Gender must be M, F, or O")
    private String gender;

    @JsonProperty("id_number")
    private String idNumber;

    @JsonProperty("phone_number")
    @Pattern(regexp = "^\\\\d{10,11}$|^$", message = "Invalid phone number")
    private String phoneNumber;

    @JsonProperty("birth_date")
    private LocalDate birthDate;

    @JsonProperty("medical_history")
    private String medicalHistory;
}
''',

    # Backend - HealthProfileDTO
    "backend/src/main/java/com/chronicdisease/dto/HealthProfileDTO.java": '''package com.chronicdisease.dto;

import com.fasterxml.jackson.annotation.JsonProperty;
import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.time.LocalDateTime;
import java.util.List;
import java.util.UUID;

@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class HealthProfileDTO {

    private UUID id;

    @JsonProperty("patient_id")
    private UUID patientId;

    @JsonProperty("risk_level")
    private String riskLevel;

    @JsonProperty("risk_score")
    private Float riskScore;

    @JsonProperty("identified_diseases")
    private List<String> identifiedDiseases;

    private List<String> recommendations;

    @JsonProperty("updated_at")
    private LocalDateTime updatedAt;
}
''',

    # Backend - PatientRepository
    "backend/src/main/java/com/chronicdisease/repository/PatientRepository.java": '''package com.chronicdisease.repository;

import com.chronicdisease.entity.Patient;
import com.chronicdisease.entity.Tenant;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

import java.util.List;
import java.util.Optional;
import java.util.UUID;

@Repository
public interface PatientRepository extends JpaRepository<Patient, UUID> {
    List<Patient> findByTenant(Tenant tenant);
    Optional<Patient> findByIdAndTenant(UUID id, Tenant tenant);
    List<Patient> findByNameContainingAndTenant(String name, Tenant tenant);
}
''',

    # Backend - HealthMetricRepository
    "backend/src/main/java/com/chronicdisease/repository/HealthMetricRepository.java": '''package com.chronicdisease.repository;

import com.chronicdisease.entity.HealthMetric;
import com.chronicdisease.entity.Patient;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

import java.time.LocalDateTime;
import java.util.List;
import java.util.UUID;

@Repository
public interface HealthMetricRepository extends JpaRepository<HealthMetric, UUID> {
    List<HealthMetric> findByPatient(Patient patient);
    List<HealthMetric> findByPatientAndMetricType(Patient patient, String metricType);
    List<HealthMetric> findByPatientAndRecordedAtBetween(Patient patient, LocalDateTime start, LocalDateTime end);
}
''',

    # Backend - TenantRepository
    "backend/src/main/java/com/chronicdisease/repository/TenantRepository.java": '''package com.chronicdisease.repository;

import com.chronicdisease.entity.Tenant;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

import java.util.UUID;

@Repository
public interface TenantRepository extends JpaRepository<Tenant, UUID> {
}
''',

    # Backend - HealthProfileRepository
    "backend/src/main/java/com/chronicdisease/repository/HealthProfileRepository.java": '''package com.chronicdisease.repository;

import com.chronicdisease.entity.HealthProfile;
import com.chronicdisease.entity.Patient;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

import java.util.Optional;
import java.util.UUID;

@Repository
public interface HealthProfileRepository extends JpaRepository<HealthProfile, UUID> {
    Optional<HealthProfile> findByPatient(Patient patient);
}
''',

    # Backend - PatientService
    "backend/src/main/java/com/chronicdisease/service/PatientService.java": '''package com.chronicdisease.service;

import com.chronicdisease.dto.PatientDTO;
import com.chronicdisease.entity.Patient;
import com.chronicdisease.entity.Tenant;
import com.chronicdisease.repository.PatientRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import javax.persistence.EntityNotFoundException;
import java.util.List;
import java.util.UUID;
import java.util.stream.Collectors;

@Service
public class PatientService {

    @Autowired
    private PatientRepository patientRepository;

    @Autowired
    private TenantService tenantService;

    public PatientDTO createPatient(PatientDTO patientDTO) {
        Tenant tenant = tenantService.getCurrentTenant();
        Patient patient = Patient.builder()
                .tenant(tenant)
                .name(patientDTO.getName())
                .age(patientDTO.getAge())
                .gender(patientDTO.getGender())
                .idNumber(patientDTO.getIdNumber())
                .phoneNumber(patientDTO.getPhoneNumber())
                .birthDate(patientDTO.getBirthDate())
                .medicalHistory(patientDTO.getMedicalHistory())
                .build();

        Patient saved = patientRepository.save(patient);
        return convertToDTO(saved);
    }

    public PatientDTO getPatient(UUID id) {
        Tenant tenant = tenantService.getCurrentTenant();
        Patient patient = patientRepository.findByIdAndTenant(id, tenant)
                .orElseThrow(() -> new EntityNotFoundException("Patient not found"));
        return convertToDTO(patient);
    }

    public List<PatientDTO> getAllPatients() {
        Tenant tenant = tenantService.getCurrentTenant();
        List<Patient> patients = patientRepository.findByTenant(tenant);
        return patients.stream().map(this::convertToDTO).collect(Collectors.toList());
    }

    public PatientDTO updatePatient(UUID id, PatientDTO patientDTO) {
        Tenant tenant = tenantService.getCurrentTenant();
        Patient patient = patientRepository.findByIdAndTenant(id, tenant)
                .orElseThrow(() -> new EntityNotFoundException("Patient not found"));

        patient.setName(patientDTO.getName());
        patient.setAge(patientDTO.getAge());
        patient.setGender(patientDTO.getGender());

        Patient updated = patientRepository.save(patient);
        return convertToDTO(updated);
    }

    public void deletePatient(UUID id) {
        Tenant tenant = tenantService.getCurrentTenant();
        Patient patient = patientRepository.findByIdAndTenant(id, tenant)
                .orElseThrow(() -> new EntityNotFoundException("Patient not found"));
        patientRepository.delete(patient);
    }

    private PatientDTO convertToDTO(Patient patient) {
        return PatientDTO.builder()
                .id(patient.getId())
                .name(patient.getName())
                .age(patient.getAge())
                .gender(patient.getGender())
                .idNumber(patient.getIdNumber())
                .phoneNumber(patient.getPhoneNumber())
                .birthDate(patient.getBirthDate())
                .medicalHistory(patient.getMedicalHistory())
                .build();
    }
}
''',

    # Backend - TenantService
    "backend/src/main/java/com/chronicdisease/service/TenantService.java": '''package com.chronicdisease.service;

import com.chronicdisease.entity.Tenant;
import com.chronicdisease.repository.TenantRepository;
import com.chronicdisease.util.TenantContext;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import javax.persistence.EntityNotFoundException;
import java.util.UUID;

@Service
public class TenantService {

    @Autowired
    private TenantRepository tenantRepository;

    public Tenant getCurrentTenant() {
        UUID tenantId = TenantContext.getTenantId();
        if (tenantId == null) {
            throw new RuntimeException("Tenant context not set");
        }
        return tenantRepository.findById(tenantId)
                .orElseThrow(() -> new EntityNotFoundException("Tenant not found"));
    }

    public Tenant getTenantById(UUID id) {
        return tenantRepository.findById(id)
                .orElseThrow(() -> new EntityNotFoundException("Tenant not found"));
    }

    public Tenant createTenant(String name) {
        Tenant tenant = Tenant.builder()
                .name(name)
                .status("ACTIVE")
                .build();
        return tenantRepository.save(tenant);
    }
}
''',

    # Backend - PatientController
    "backend/src/main/java/com/chronicdisease/controller/PatientController.java": '''package com.chronicdisease.controller;

import com.chronicdisease.dto.PatientDTO;
import com.chronicdisease.service.PatientService;
import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.tags.Tag;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import javax.validation.Valid;
import java.util.List;
import java.util.UUID;

@RestController
@RequestMapping("/api/v1/patients")
@Tag(name = "Patient Management", description = "Patient management APIs")
public class PatientController {

    @Autowired
    private PatientService patientService;

    @PostMapping
    @Operation(summary = "Create a new patient")
    public ResponseEntity<PatientDTO> createPatient(@Valid @RequestBody PatientDTO patientDTO) {
        PatientDTO created = patientService.createPatient(patientDTO);
        return ResponseEntity.status(HttpStatus.CREATED).body(created);
    }

    @GetMapping("/{id}")
    @Operation(summary = "Get patient by ID")
    public ResponseEntity<PatientDTO> getPatient(@PathVariable UUID id) {
        PatientDTO patient = patientService.getPatient(id);
        return ResponseEntity.ok(patient);
    }

    @GetMapping
    @Operation(summary = "Get all patients")
    public ResponseEntity<List<PatientDTO>> getAllPatients() {
        List<PatientDTO> patients = patientService.getAllPatients();
        return ResponseEntity.ok(patients);
    }

    @PutMapping("/{id}")
    @Operation(summary = "Update patient")
    public ResponseEntity<PatientDTO> updatePatient(
            @PathVariable UUID id,
            @Valid @RequestBody PatientDTO patientDTO) {
        PatientDTO updated = patientService.updatePatient(id, patientDTO);
        return ResponseEntity.ok(updated);
    }

    @DeleteMapping("/{id}")
    @Operation(summary = "Delete patient")
    public ResponseEntity<Void> deletePatient(@PathVariable UUID id) {
        patientService.deletePatient(id);
        return ResponseEntity.noContent().build();
    }
}
''',

    # Backend - TenantFilter
    "backend/src/main/java/com/chronicdisease/filter/TenantFilter.java": '''package com.chronicdisease.filter;

import com.chronicdisease.util.TenantContext;
import org.springframework.stereotype.Component;
import org.springframework.web.filter.OncePerRequestFilter;

import javax.servlet.FilterChain;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.util.UUID;

@Component
public class TenantFilter extends OncePerRequestFilter {

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response,
                                    FilterChain filterChain) throws ServletException, IOException {
        try {
            String tenantIdHeader = request.getHeader("X-Tenant-ID");
            if (tenantIdHeader != null && !tenantIdHeader.isEmpty()) {
                TenantContext.setTenantId(UUID.fromString(tenantIdHeader));
            }
            filterChain.doFilter(request, response);
        } finally {
            TenantContext.clear();
        }
    }
}
''',

    # Backend - application.yml
    "backend/src/main/resources/application.yml": '''spring:
  application:
    name: chronic-disease-health-profile-saas

  datasource:
    url: jdbc:postgresql://${DB_HOST:localhost}:${DB_PORT:5432}/${DB_NAME:chronic_disease_db}
    username: ${DB_USER:postgres}
    password: ${DB_PASSWORD:postgres}
    driver-class-name: org.postgresql.Driver

  jpa:
    hibernate:
      ddl-auto: validate
    properties:
      hibernate:
        dialect: org.hibernate.dialect.PostgreSQL95Dialect
        format_sql: true

  redis:
    host: ${REDIS_HOST:localhost}
    port: ${REDIS_PORT:6379}
    password: ${REDIS_PASSWORD:}

  cache:
    type: redis

server:
  port: ${SERVER_PORT:8080}
  servlet:
    context-path: /api

springdoc:
  swagger-ui:
    path: /swagger-ui.html
    enabled: true
  api-docs:
    path: /v3/api-docs
''',

    # ==================== FRONTEND FILES ====================

    # Frontend - package.json
    "frontend/package.json": '''{
  "name": "chronic-disease-health-profile-ui",
  "version": "1.0.0",
  "description": "Frontend for chronic disease health profile SaaS",
  "private": true,
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "react-router-dom": "^6.14.0",
    "@reduxjs/toolkit": "^1.9.5",
    "react-redux": "^8.1.2",
    "axios": "^1.4.0",
    "antd": "^5.8.0",
    "@ant-design/icons": "^5.1.0",
    "echarts": "^5.4.2",
    "echarts-for-react": "^3.0.2",
    "dayjs": "^1.11.9",
    "lodash": "^4.17.21",
    "classnames": "^2.3.2"
  },
  "devDependencies": {
    "react-scripts": "5.0.1",
    "@types/react": "^18.2.0",
    "@types/react-dom": "^18.2.0",
    "@types/node": "^20.3.0",
    "typescript": "^5.1.6"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test",
    "eject": "react-scripts eject"
  },
  "eslintConfig": {
    "extends": ["react-app"]
  },
  "browserslist": {
    "production": [">0.2%", "not dead", "not op_mini all"],
    "development": ["last 1 chrome version", "last 1 firefox version", "last 1 safari version"]
  }
}
''',

    # Frontend - tsconfig.json
    "frontend/tsconfig.json": '''{
  "compilerOptions": {
    "target": "ES2020",
    "useDefineForClassFields": true,
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "skipLibCheck": true,
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "strict": true,
    "resolveJsonModule": true,
    "moduleResolution": "node",
    "allowJs": true,
    "jsx": "react-jsx"
  },
  "include": ["src"],
  "exclude": ["node_modules"]
}
''',

    # Frontend - public/index.html
    "frontend/public/index.html": '''<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <meta name="theme-color" content="#000000" />
    <meta name="description" content="Chronic Disease Health Profile SaaS Platform" />
    <title>慢病健康画像中台</title>
</head>
<body>
    <noscript>You need to enable JavaScript to run this app.</noscript>
    <div id="root"></div>
</body>
</html>
''',

    # Frontend - src/index.tsx
    "frontend/src/index.tsx": '''import React from 'react';
import ReactDOM from 'react-dom/client';
import { Provider } from 'react-redux';
import store from './store/store';
import App from './App';
import './styles/globals.css';

const root = ReactDOM.createRoot(
  document.getElementById('root') as HTMLElement
);

root.render(
  <React.StrictMode>
    <Provider store={store}>
      <App />
    </Provider>
  </React.StrictMode>
);
''',

    # Frontend - src/App.tsx
    "frontend/src/App.tsx": '''import React from 'react';
import { BrowserRouter as Router, Routes, Route } from 'react-router-dom';
import Login from './components/Auth/Login';
import ProtectedRoute from './components/Auth/ProtectedRoute';
import Layout from './components/Layout/Layout';
import HomePage from './pages/HomePage';
import PatientManagement from './pages/PatientManagement';
import NotFound from './pages/NotFound';

const App: React.FC = () => {
  return (
    <Router>
      <Routes>
        <Route path="/login" element={<Login />} />
        
        <Route element={<ProtectedRoute />}>
          <Route element={<Layout />}>
            <Route path="/" element={<HomePage />} />
            <Route path="/patients" element={<PatientManagement />} />
          </Route>
        </Route>

        <Route path="*" element={<NotFound />} />
      </Routes>
    </Router>
  );
};

export default App;
''',

    # Frontend - src/store/store.ts
    "frontend/src/store/store.ts": '''import { configureStore } from '@reduxjs/toolkit';
import authReducer from './slices/authSlice';
import patientReducer from './slices/patientSlice';
import uiReducer from './slices/uiSlice';

const store = configureStore({
  reducer: {
    auth: authReducer,
    patient: patientReducer,
    ui: uiReducer,
  },
});

export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;

export default store;
''',

    # Frontend - src/store/hooks.ts
    "frontend/src/store/hooks.ts": '''import { TypedUseSelectorHook, useDispatch, useSelector } from 'react-redux';
import type { RootState, AppDispatch } from './store';

export const useAppDispatch = () => useDispatch<AppDispatch>();
export const useAppSelector: TypedUseSelectorHook<RootState> = useSelector;
''',

    # Frontend - src/store/slices/authSlice.ts
    "frontend/src/store/slices/authSlice.ts": '''import { createSlice, PayloadAction } from '@reduxjs/toolkit';

interface AuthState {
  isAuthenticated: boolean;
  user: any | null;
  token: string | null;
}

const initialState: AuthState = {
  isAuthenticated: !!localStorage.getItem('token'),
  user: null,
  token: localStorage.getItem('token'),
};

const authSlice = createSlice({
  name: 'auth',
  initialState,
  reducers: {
    loginSuccess: (state, action: PayloadAction<{ token: string; user: any }>) => {
      state.isAuthenticated = true;
      state.token = action.payload.token;
      state.user = action.payload.user;
      localStorage.setItem('token', action.payload.token);
    },
    logout: (state) => {
      state.isAuthenticated = false;
      state.token = null;
      state.user = null;
      localStorage.removeItem('token');
    },
  },
});

export const { loginSuccess, logout } = authSlice.actions;
export default authSlice.reducer;
''',

    # Frontend - src/store/slices/patientSlice.ts
    "frontend/src/store/slices/patientSlice.ts": '''import { createSlice, PayloadAction } from '@reduxjs/toolkit';

interface Patient {
  id: string;
  name: string;
  age: number;
  gender: string;
}

interface PatientState {
  patients: Patient[];
  currentPatient: Patient | null;
  loading: boolean;
}

const initialState: PatientState = {
  patients: [],
  currentPatient: null,
  loading: false,
};

const patientSlice = createSlice({
  name: 'patient',
  initialState,
  reducers: {
    setPatients: (state, action: PayloadAction<Patient[]>) => {
      state.patients = action.payload;
    },
    setCurrentPatient: (state, action: PayloadAction<Patient | null>) => {
      state.currentPatient = action.payload;
    },
    addPatient: (state, action: PayloadAction<Patient>) => {
      state.patients.push(action.payload);
    },
  },
});

export const { setPatients, setCurrentPatient, addPatient } = patientSlice.actions;
export default patientSlice.reducer;
''',

    # Frontend - src/store/slices/uiSlice.ts
    "frontend/src/store/slices/uiSlice.ts": '''import { createSlice, PayloadAction } from '@reduxjs/toolkit';

interface UIState {
  sidebarOpen: boolean;
}

const initialState: UIState = {
  sidebarOpen: true,
};

const uiSlice = createSlice({
  name: 'ui',
  initialState,
  reducers: {
    toggleSidebar: (state) => {
      state.sidebarOpen = !state.sidebarOpen;
    },
  },
});

export const { toggleSidebar } = uiSlice.actions;
export default uiSlice.reducer;
''',

    # Frontend - src/services/api.ts
    "frontend/src/services/api.ts": '''import axios from 'axios';

const API_BASE_URL = process.env.REACT_APP_API_BASE_URL || 'http://localhost:8080/api';

const api = axios.create({
  baseURL: API_BASE_URL,
  timeout: 10000,
});

api.interceptors.request.use((config) => {
  const token = localStorage.getItem('token');
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }

  const tenantId = localStorage.getItem('tenantId');
  if (tenantId) {
    config.headers['X-Tenant-ID'] = tenantId;
  }

  return config;
});

export default api;
''',

    # Frontend - src/components/Layout/Layout.tsx
    "frontend/src/components/Layout/Layout.tsx": '''import React, { useState } from 'react';
import { Layout, Menu, Button } from 'antd';
import { MenuFoldOutlined, MenuUnfoldOutlined } from '@ant-design/icons';
import { useNavigate } from 'react-router-dom';
import { Outlet } from 'react-router-dom';
import './Layout.css';

const { Header, Sider, Content } = Layout;

const LayoutComponent: React.FC = () => {
  const [collapsed, setCollapsed] = useState(false);
  const navigate = useNavigate();

  return (
    <Layout style={{ minHeight: '100vh' }}>
      <Sider trigger={null} collapsible collapsed={collapsed}>
        <div className="logo">
          <h2>{collapsed ? '慢病' : '慢病健康画像中台'}</h2>
        </div>
        <Menu
          theme="dark"
          mode="inline"
          items={[
            {
              key: '/',
              label: 'Dashboard',
              onClick: () => navigate('/'),
            },
            {
              key: '/patients',
              label: 'Patients',
              onClick: () => navigate('/patients'),
            },
          ]}
        />
      </Sider>
      <Layout>
        <Header style={{ background: '#fff', padding: '0 16px' }}>
          <Button
            type="text"
            icon={collapsed ? <MenuUnfoldOutlined /> : <MenuFoldOutlined />}
            onClick={() => setCollapsed(!collapsed)}
            style={{ fontSize: '16px' }}
          />
        </Header>
        <Content style={{ margin: '24px 16px', padding: 24, background: '#fff' }}>
          <Outlet />
        </Content>
      </Layout>
    </Layout>
  );
};

export default LayoutComponent;
''',

    # Frontend - src/components/Layout/Layout.css
    "frontend/src/components/Layout/Layout.css": '''.logo {
  height: 64px;
  background: rgba(255, 255, 255, 0.2);
  margin: 16px;
  border-radius: 6px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.logo h2 {
  color: white;
  margin: 0;
  font-size: 16px;
}
''',

    # Frontend - src/components/Auth/Login.tsx
    "frontend/src/components/Auth/Login.tsx": '''import React, { useState } from 'react';
import { Form, Input, Button, Card, message } from 'antd';
import { useNavigate } from 'react-router-dom';
import { useAppDispatch } from '../../store/hooks';
import { loginSuccess } from '../../store/slices/authSlice';
import './Auth.css';

const Login: React.FC = () => {
  const [loading, setLoading] = useState(false);
  const navigate = useNavigate();
  const dispatch = useAppDispatch();
  const [form] = Form.useForm();

  const onFinish = async (values: any) => {
    setLoading(true);
    try {
      const mockToken = 'mock-jwt-token-' + Date.now();
      const mockUser = {
        id: '1',
        username: values.username,
        tenantId: 'default-tenant',
      };

      localStorage.setItem('tenantId', mockUser.tenantId);
      dispatch(loginSuccess({ token: mockToken, user: mockUser }));
      message.success('Login successful!');
      navigate('/');
    } catch (error) {
      message.error('Login failed!');
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="login-container">
      <Card className="login-card">
        <h1 className="login-title">慢病健康画像中台</h1>
        <Form form={form} onFinish={onFinish}>
          <Form.Item
            name="username"
            rules={[{ required: true, message: 'Please enter username' }]}
          >
            <Input placeholder="Username" />
          </Form.Item>
          <Form.Item
            name="password"
            rules={[{ required: true, message: 'Please enter password' }]}
          >
            <Input.Password placeholder="Password" />
          </Form.Item>
          <Form.Item>
            <Button type="primary" block loading={loading} htmlType="submit">
              Login
            </Button>
          </Form.Item>
        </Form>
      </Card>
    </div>
  );
};

export default Login;
''',

    # Frontend - src/components/Auth/Auth.css
    "frontend/src/components/Auth/Auth.css": '''.login-container {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
}

.login-card {
  width: 100%;
  max-width: 400px;
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.15);
}

.login-title {
  text-align: center;
  margin-bottom: 30px;
  color: #333;
}
''',

    # Frontend - src/components/Auth/ProtectedRoute.tsx
    "frontend/src/components/Auth/ProtectedRoute.tsx": '''import React from 'react';
import { Navigate, Outlet } from 'react-router-dom';
import { useAppSelector } from '../../store/hooks';

const ProtectedRoute: React.FC = () => {
  const { isAuthenticated } = useAppSelector((state) => state.auth);

  return isAuthenticated ? <Outlet /> : <Navigate to="/login" />;
};

export default ProtectedRoute;
''',

    # Frontend - src/pages/HomePage.tsx
    "frontend/src/pages/HomePage.tsx": '''import React, { useEffect, useState } from 'react';
import { Row, Col, Card, Statistic } from 'antd';

const HomePage: React.FC = () => {
  const [stats, setStats] = useState({
    totalPatients: 0,
    highRiskPatients: 0,
  });

  useEffect(() => {
    setStats({
      totalPatients: 150,
      highRiskPatients: 30,
    });
  }, []);

  return (
    <div>
      <h1>Dashboard</h1>
      <Row gutter={16} style={{ marginBottom: '24px' }}>
        <Col xs={24} sm={12}>
          <Card>
            <Statistic title="Total Patients" value={stats.totalPatients} />
          </Card>
        </Col>
        <Col xs={24} sm={12}>
          <Card>
            <Statistic title="High Risk" value={stats.highRiskPatients} />
          </Card>
        </Col>
      </Row>
    </div>
  );
};

export default HomePage;
''',

    # Frontend - src/pages/PatientManagement.tsx
    "frontend/src/pages/PatientManagement.tsx": '''import React, { useState } from 'react';
import { Table, Button, Space, Modal, Form, Input, Select, message } from 'antd';

const PatientManagement: React.FC = () => {
  const [patients, setPatients] = useState<any[]>([]);
  const [isModalOpen, setIsModalOpen] = useState(false);
  const [form] = Form.useForm();

  const handleAddPatient = async (values: any) => {
    try {
      setPatients([...patients, { id: Date.now(), ...values }]);
      message.success('Patient added successfully!');
      setIsModalOpen(false);
      form.resetFields();
    } catch (error) {
      message.error('Failed to add patient');
    }
  };

  const columns = [
    { title: 'Name', dataIndex: 'name', key: 'name' },
    { title: 'Age', dataIndex: 'age', key: 'age' },
    { title: 'Gender', dataIndex: 'gender', key: 'gender' },
  ];

  return (
    <div>
      <h1>Patient Management</h1>
      <Button type="primary" onClick={() => setIsModalOpen(true)} style={{ marginBottom: '16px' }}>
        Add Patient
      </Button>

      <Table columns={columns} dataSource={patients} rowKey="id" />

      <Modal title="Add Patient" open={isModalOpen} onCancel={() => setIsModalOpen(false)} footer={null}>
        <Form form={form} onFinish={handleAddPatient}>
          <Form.Item name="name" rules={[{ required: true }]}>
            <Input placeholder="Name" />
          </Form.Item>
          <Form.Item name="age" rules={[{ required: true }]}>
            <Input type="number" placeholder="Age" />
          </Form.Item>
          <Form.Item name="gender" rules={[{ required: true }]}>
            <Select placeholder="Gender" options={[
              { label: 'Male', value: 'M' },
              { label: 'Female', value: 'F' },
            ]} />
          </Form.Item>
          <Button type="primary" htmlType="submit" block>
            Add
          </Button>
        </Form>
      </Modal>
    </div>
  );
};

export default PatientManagement;
''',

    # Frontend - src/pages/NotFound.tsx
    "frontend/src/pages/NotFound.tsx": '''import React from 'react';
import { Button, Result } from 'antd';
import { useNavigate } from 'react-router-dom';

const NotFound: React.FC = () => {
  const navigate = useNavigate();

  return (
    <Result
      status="404"
      title="404"
      subTitle="Sorry, the page you visited does not exist."
      extra={<Button type="primary" onClick={() => navigate('/')}>Back Home</Button>}
    />
  );
};

export default NotFound;
''',

    # Frontend - src/styles/globals.css
    "frontend/src/styles/globals.css": '''* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', 'Roboto', 'Oxygen',
    'Ubuntu', 'Cantarell', 'Fira Sans', 'Droid Sans',
    'Helvetica Neue', sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  background-color: #f5f5f5;
}
''',

    # Frontend - .env.example
    "frontend/.env.example": '''REACT_APP_API_BASE_URL=http://localhost:8080/api
REACT_APP_TENANT_ID=default-tenant
''',

    # Frontend - Dockerfile
    "frontend/Dockerfile": '''FROM node:18-alpine as build
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

FROM node:18-alpine
WORKDIR /app
RUN npm install -g serve
COPY --from=build /app/build ./build
EXPOSE 3000
CMD ["serve", "-s", "build", "-l", "3000"]
''',

    # ==================== ML SERVICE FILES ====================

    # ML Service - requirements.txt
    "ml-service/requirements.txt": '''Flask==2.3.2
Flask-CORS==4.0.0
numpy==1.24.3
pandas==2.0.3
scikit-learn==1.2.2
tensorflow==2.12.0
joblib==1.2.0
python-dotenv==1.0.0
gunicorn==20.1.0
requests==2.31.0
''',

    # ML Service - config.py
    "ml-service/config.py": '''import os
from dotenv import load_dotenv

load_dotenv()

class Config:
    FLASK_ENV = os.getenv('FLASK_ENV', 'development')
    DEBUG = FLASK_ENV == 'development'
    MODEL_PATH = os.getenv('MODEL_PATH', './ml_models/risk_model.pkl')
    LOG_LEVEL = os.getenv('LOG_LEVEL', 'INFO')

config = Config()
''',

    # ML Service - app.py
    "ml-service/app.py": '''from flask import Flask, jsonify, request
from flask_cors import CORS
import logging
from config import config

logging.basicConfig(level=config.LOG_LEVEL)
logger = logging.getLogger(__name__)

app = Flask(__name__)
CORS(app)

@app.route('/api/health/check', methods=['GET'])
def health_check():
    return jsonify({'status': 'healthy'}), 200

@app.route('/api/predict', methods=['POST'])
def predict():
    try:
        data = request.get_json()
        features = data.get('features', {})
        
        # Mock prediction logic
        risk_score = 50.0
        
        return jsonify({
            'risk_score': risk_score,
            'risk_level': 'MEDIUM',
            'confidence': 85.0
        }), 200
    except Exception as e:
        logger.error(f'Prediction error: {str(e)}')
        return jsonify({'error': str(e)}), 500

@app.errorhandler(404)
def not_found(error):
    return jsonify({'error': 'Not found'}), 404

@app.errorhandler(500)
def internal_error(error):
    logger.error(f'Internal error: {error}')
    return jsonify({'error': 'Internal server error'}), 500

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000, debug=config.DEBUG)
''',

    # ML Service - models/risk_predictor.py
    "ml-service/models/risk_predictor.py": '''import numpy as np
from sklearn.ensemble import RandomForestClassifier
import joblib
import os

class RiskPredictor:
    def __init__(self, model_path='./ml_models/risk_model.pkl'):
        self.model_path = model_path
        self.model = None
        self.feature_names = ['age', 'bmi', 'blood_glucose', 'sys_bp', 'cholesterol']
        self.load_model()

    def load_model(self):
        if os.path.exists(self.model_path):
            self.model = joblib.load(self.model_path)
        else:
            self.model = RandomForestClassifier(n_estimators=100, random_state=42)

    def predict_risk(self, patient_data):
        try:
            features = []
            for feature in self.feature_names:
                value = patient_data.get(feature, 0)
                features.append(float(value))
            
            features = np.array(features).reshape(1, -1)
            risk_probs = self.model.predict_proba(features)
            risk_score = float(risk_probs[0][1] * 100)
            
            if risk_score >= 70:
                risk_level = "HIGH"
            elif risk_score >= 40:
                risk_level = "MEDIUM"
            else:
                risk_level = "LOW"
            
            return {
                'risk_score': round(risk_score, 2),
                'risk_level': risk_level,
                'confidence': round(max(risk_probs[0]) * 100, 2)
            }
        except Exception as e:
            raise ValueError(f"Error predicting risk: {str(e)}")
''',

    # ML Service - Dockerfile
    "ml-service/Dockerfile": '''FROM python:3.10-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
EXPOSE 5000
CMD ["gunicorn", "--bind", "0.0.0.0:5000", "--workers", "4", "--timeout", "120", "app:app"]
''',

    # ==================== CONFIG FILES ====================

    # docker-compose.yml
    "docker-compose.yml": '''version: '3.8'

services:
  postgres:
    image: postgres:15-alpine
    container_name: chronic_disease_db
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: chronic_disease_db
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - chronic-disease-network
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    container_name: chronic_disease_redis
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    networks:
      - chronic-disease-network
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5

  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
    container_name: chronic_disease_backend
    environment:
      DB_HOST: postgres
      DB_PORT: 5432
      DB_NAME: chronic_disease_db
      DB_USER: postgres
      DB_PASSWORD: postgres
      REDIS_HOST: redis
      REDIS_PORT: 6379
      SERVER_PORT: 8080
    ports:
      - "8080:8080"
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
    networks:
      - chronic-disease-network

  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
    container_name: chronic_disease_frontend
    ports:
      - "3000:3000"
    environment:
      REACT_APP_API_BASE_URL: http://localhost:8080/api
    depends_on:
      - backend
    networks:
      - chronic-disease-network

  ml-service:
    build:
      context: ./ml-service
      dockerfile: Dockerfile
    container_name: chronic_disease_ml
    environment:
      FLASK_ENV: production
    ports:
      - "5000:5000"
    networks:
      - chronic-disease-network
    volumes:
      - ./ml-service/ml_models:/app/ml_models

volumes:
  postgres_data:
  redis_data:

networks:
  chronic-disease-network:
    driver: bridge
''',

    # .gitignore
    ".gitignore": '''# Backend
backend/target/
backend/.classpath
backend/.project
backend/.settings/
backend/bin/
backend/*.log

# Frontend
frontend/node_modules/
frontend/build/
frontend/dist/
frontend/.env.local
frontend/.env.*.local
frontend/npm-debug.log*
frontend/yarn-debug.log*
frontend/yarn-error.log*

# ML Service
ml-service/__pycache__/
ml-service/*.py[cod]
ml-service/*$py.class
ml-service/*.egg-info/
ml-service/venv/
ml-service/.Python
ml-service/ml_models/*.pkl

# IDE
.vscode/
.idea/
*.swp
*.swo
*~
.DS_Store

.env
.env.local
''',

    # README.md
    "README.md": '''# Chronic Disease Health Profile SaaS

A comprehensive SaaS platform for identifying and building health profiles for patients with chronic diseases using machine learning and advanced analytics.

## 🎯 Overview

This platform provides:
- **Patient Management**: Complete patient information management system
- **Health Metrics Tracking**: Real-time health metric data collection
- **Risk Assessment**: ML-powered chronic disease risk prediction
- **Health Profiles**: Personalized health profiles with recommendations
- **Multi-Tenant Architecture**: Secure, isolated data for multiple healthcare organizations

## 🚀 Quick Start

### Prerequisites
- Docker & Docker Compose
- Git

### Installation

1. Clone the repository:
```bash
git clone https://github.com/karry921111111-maker/chronic-disease-health-profile-saas.git
cd chronic-disease-health-profile-saas
