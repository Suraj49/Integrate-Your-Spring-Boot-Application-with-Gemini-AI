# Integrate-Your-Spring-Boot-Application-with-Gemini-AI

This Spring Boot application demonstrates how to integrate **Gemini AI** using **RestTemplate**. By following this guide, you'll learn how to connect your application with Gemini AI's API, enabling advanced AI capabilities such as natural language processing and machine learning.


## Table of Contents

1. [Configure Dependencies](#Configure-Dependencies)
2. [API Credentials](#API-Credentials)
3. [Integration with RestTemplate](#Integration-with-RestTemplate)
4. [CURL](#CURL)

## Configure Dependencies

### Configure Dependencies

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```
## API Credentials

```properties
gemini.api.url=https://generativelanguage.googleapis.com/v1beta/models/gemini-1.5-flash:generateContent?key=your_api_key
```

## Integration with RestTemplate

## GeminiService Class.

```java
import org.springframework.beans.factory.annotation.Value;
import org.springframework.http.HttpEntity;
import org.springframework.http.HttpHeaders;
import org.springframework.http.HttpMethod;
import org.springframework.http.ResponseEntity;
import org.springframework.stereotype.Service;
import org.springframework.web.client.RestTemplate;

@Service
public class GeminiAIService {

    @Value("${gemini.api.url}")
    private String apiUrl;

    private final RestTemplate restTemplate;

    public GeminiAIService(RestTemplate restTemplate) {
        this.restTemplate = restTemplate;
    }

     public Object callGeminiAPI(Prompt prompt){

        HttpEntity<Prompt> requestEntity = new HttpEntity<>(prompt);

        ResponseEntity<Object> response = restTemplate.exchange(
                apiUrl,
                HttpMethod.POST,
                requestEntity,
                Object.class
        );
        return response.getBody();
    }
}
```

## Conroller Class

```Java
package com.gemini.controller;


import com.gemini.dto.Contents;
import com.gemini.dto.Parts;
import com.gemini.dto.Prompt;
import com.gemini.service.AppService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.ArrayList;
import java.util.List;

@RestController
@RequestMapping("/api")
@CrossOrigin("http://localhost:4200")
public class ApiController {

    @Autowired
    AppService appService;

    @PostMapping("/get-result")
    ResponseEntity<Object> getResponseFromGemini(@RequestBody Parts parts){

        Prompt prompt = new Prompt();
        Contents contents = new Contents();

        List<Contents> contentsList = new ArrayList<>();
        List<Parts> partsList = new ArrayList<>();

        partsList.add(parts);

        contents.setParts(partsList);

        contentsList.add(contents);
        prompt.setContents(contentsList);


        return new ResponseEntity<>(appService.getResult(prompt, parts), HttpStatus.OK);
    }
}

```
## AppService Class

```java
package com.gemini.service;

import com.gemini.dto.Parts;
import com.gemini.dto.Prompt;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.stereotype.Service;

import java.io.FileWriter;
import java.io.IOException;
import java.util.List;
import java.util.Map;

@Service
public class AppService {

    @Autowired
    GeminiAIService geminiService;

    public ResponseEntity<Object> getResult(Prompt prompt, Parts parts){
        Object response = geminiService.callGeminiAPI(prompt);
        return new ResponseEntity<>(response, HttpStatus.OK);
    }

}
```
## POJO Class

```java
package com.gemini.dto;

import lombok.Data;

import java.util.List;

@Data
public class Contents {
    List<Parts> parts;
}

package com.gemini.dto;

import lombok.Data;

@Data
public class Parts {
    String text;
}


package com.gemini.dto;

import lombok.Data;

import java.util.List;

@Data
public class Prompt {
    List<Contents> contents;
}

```

## CURL
```bash
curl --location --request POST 'http://localhost:8080/api/get-result' \
--header 'Content-Type: application/json' \
--data '{
   "text": "Hello."
}'
```



