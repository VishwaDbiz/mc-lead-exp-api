### Business Requirement: Lead Change Event Listener
**Requirement ID:** BR-001  
**Endpoint:** POST https://mc-lead-exp-kd7ppn.9675fv.sgp-s1.cloudhub.io/Lead Change Event Listener  

**Business Objective**  
The Lead Change Event Listener is designed to capture and process lead change events from the Salesforce platform. This functionality enables real-time updates to lead information, ensuring that the sales team has access to the most current data. By integrating with Anypoint MQ, the system can efficiently handle and distribute lead change notifications across various services.

**API Contract**  
| Field | Type | Required | Description |
| --- | --- | --- | --- |
| No request schema available |  |  |  |

**Response Schema**  
| HTTP Status | Field | Type | Description |
| --- | --- | --- | --- |
| No response schema available |  |  |  |

**Flow Implementation**  
1. The system shall be triggered by Salesforce platform events on the channel: "/data/LeadChangeEvent".
2. The system shall log a message at INFO level indicating the receipt of a lead change event.
3. The system shall transform the payload using DataWeave to extract the relevant lead information.
4. The system shall publish a message to an Anypoint MQ queue/topic with the transformed lead data.
5. The system shall log a message at INFO level confirming the successful publication of the message.
6. The system shall handle any errors raised within the flow during processing.
7. The system shall continue processing on error and return a handled response to ensure stability.

**Field Mapping / Data Transformation**  
| Source Field | Target Field | Transformation Logic |
| --- | --- | --- |
| payload | payload | The system extracts the 'payload' field from the incoming payload structure. |

**Connected Systems**  
| System | Protocol | URL / Destination | Direction | Purpose |
| --- | --- | --- | --- | --- |
| Anypoint MQ | anypoint-mq:publish | Anypoint_MQ_Config | Outbound | To publish lead change notifications for further processing. |

---

### Business Requirement: Retrieve Lead Information
**Requirement ID:** BR-001  
**Endpoint:** GET https://mc-lead-exp-kd7ppn.9675fv.sgp-s1.cloudhub.io/leads(leadId)

**Business Objective**  
The purpose of this API is to retrieve detailed information about a specific lead identified by the `leadId`. This functionality is essential for sales and marketing teams to access lead data efficiently, enabling them to make informed decisions and enhance customer engagement.

**API Contract**

| Field       | Type   | Required | Description                          |
|-------------|--------|----------|--------------------------------------|
| leadId      | string | Yes      | The unique identifier for the lead. |

**Response Schema**

| HTTP Status | Field       | Type   | Description                          |
|-------------|-------------|--------|--------------------------------------|
| 200         | payload     | object | The lead information retrieved.      |
| 400         | error       | string | Error message for bad requests.     |
| 404         | error       | string | Error message when lead is not found. |
| 500         | error       | string | Error message for server errors.    |

**Flow Implementation**
1. The system shall log a message at INFO level.
2. The system shall transform the payload using DataWeave, setting the method to "GET", the URL to "https://mc-lead-prc-api-kd7ppn.9675fv.sgp-s1.cloudhub.io/api/leads", the content type to "application/json", the client secret to "bb1da774897f4271842F4379B7Eb44E2", and the client ID to "99d58299e759411aac621988a467c7b9".
3. The system shall invoke the sub-flow "commonRequestSub_Flow".
4. The system shall log a message at INFO level within the sub-flow.
5. The system shall execute a try block to handle potential errors.
6. The system shall call the external REST API using the method and URL defined in the previous transformation.
7. The system shall handle any errors raised within the flow.
8. The system shall propagate the failure to the caller if an error occurs.
9. The system shall log a message at INFO level after the external API call.
10. The system shall transform the payload using DataWeave to prepare the final response.
11. The system shall log a message at INFO level after the transformation.

**Field Mapping / Data Transformation**

| Source Field                     | Target Field       | Transformation Logic                                      |
|----------------------------------|--------------------|----------------------------------------------------------|
| p('prc.get.method')             | method              | Direct mapping                                           |
| p('prc.url') ++ "/" ++ attributes.uriParams.'leadId' | url                 | Concatenation of base URL and leadId                    |
| p('prc.contentType')            | Content-Type       | Direct mapping                                           |
| p('prc.client_secret')          | client_secret      | Direct mapping                                           |
| p('prc.client_id')              | client_id          | Direct mapping                                           |
| payload                          | payload            | Direct mapping                                           |

**Connected Systems**

| System                | Protocol      | URL / Destination                                                       | Direction | Purpose                                      |
|-----------------------|---------------|------------------------------------------------------------------------|-----------|----------------------------------------------|
| External REST API     | http:request  | https://mc-lead-prc-api-kd7ppn.9675fv.sgp-s1.cloudhub.io/api/leads   | Outbound  | To retrieve lead information based on leadId |

---

### Business Requirement: Create Lead in CRM
**Requirement ID:** BR-001  
**Endpoint:** POST https://mc-lead-exp-kd7ppn.9675fv.sgp-s1.cloudhub.io/leads  

**Business Objective**  
The purpose of this API is to create a new lead in the CRM system by capturing essential information such as the lead's name, email, phone number, and company. This functionality is crucial for enhancing customer relationship management and ensuring that potential leads are effectively tracked and followed up.

**API Contract**

| Field      | Type   | Required | Description |
|------------|--------|----------|-------------|
| firstName  | string | Yes      | The first name of the lead. |
| lastName   | string | Yes      | The last name of the lead. |
| email      | string | Yes      | The email address of the lead. |
| phone      | string | Yes      | The phone number of the lead. |
| company    | string | Yes      | The company name associated with the lead. |

**Response Schema**

| HTTP Status | Field | Type   | Description |
|-------------|-------|--------|-------------|
| 201         | None  | N/A    | Indicates that the lead was successfully created. |

**Flow Implementation**
1. The system shall log a message at INFO level indicating the start of the lead creation process.
2. The system shall transform the payload using DataWeave, setting the method to "POST", the URL to "https://mc-lead-prc-api-kd7ppn.9675fv.sgp-s1.cloudhub.io/api/leads", the content type to "application/json", and including the client secret and client ID.
3. The system shall invoke the sub-flow "commonRequestSub_Flow".
4.   ┌─ Sub-flow: commonRequestSub_Flow
5.   │ The system shall log a message at INFO level indicating the start of the sub-flow.
6.   │ The system shall execute a try block to handle potential errors.
7.   │   The system shall call the external REST API using the method and URL defined in the variables.
8.   │   The system shall handle any errors raised within the flow.
9.   │     On error, the system shall propagate the failure to the caller.
10.   │ The system shall log a message at INFO level indicating the end of the sub-flow.
11.   └─ End: commonRequestSub_Flow
12. The system shall transform the payload again using DataWeave to ensure the final output is in the correct format.
13. The system shall log a message at INFO level indicating the completion of the lead creation process.

**Field Mapping / Data Transformation**

| Source Field                | Target Field                | Transformation Logic                          |
|-----------------------------|-----------------------------|-----------------------------------------------|
| p('prc.create.method')      | method                      | Direct mapping to the HTTP method "POST".    |
| p('prc.url')                | url                         | Direct mapping to the API endpoint URL.       |
| p('prc.contentType')        | Content-Type                | Direct mapping to the content type.           |
| p('prc.client_secret')      | client_secret               | Direct mapping to the client secret.          |
| p('prc.client_id')          | client_id                   | Direct mapping to the client ID.              |
| payload                      | payload                     | Direct mapping of the entire payload object.  |

**Connected Systems**

| System               | Protocol      | URL / Destination                                                       | Direction | Purpose                                      |
|----------------------|---------------|------------------------------------------------------------------------|-----------|----------------------------------------------|
| External REST API    | http:request  | https://mc-lead-prc-api-kd7ppn.9675fv.sgp-s1.cloudhub.io/api/leads   | Outbound  | To create a new lead in the CRM system.     |