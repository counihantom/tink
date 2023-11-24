# Internal Tinkerbell design

## Goal

This technobyte is an attempt to document the internal innerworkings of the Tink microservice.

Note to reader - as the code moves fast, be aware that this may be out of date in places.

But in theory, this should give a reader some additional info to understanding how Tink is plumbed.

## Service Definition

Tink exposes its services to hardware via the grpc interface `workflow.proto` defined [here](../internal/proto/workflow/v2/workflow.proto)

The source of truth is the referenced protobuf file, here we try and abstract the salient aspects

``` mermaid

---
title: Interface Diagram for Tink Workflow Service
---

classDiagram
    note "Add a note for overall diagram"
 
    WorkflowService ..> GetWorkflowsRequest
    WorkflowService ..> GetWorkflowsResponse
    WorkflowService ..> PublishEventRequest
    WorkflowService ..> PublishEventResponse
    GetWorkflowsResponse ..> cmd
    Workflow *-- "0..n" Action
    PublishEventRequest ..> Event
    cmd ..> StartWorkflow
    cmd ..> StopWorkflow
    Event ..> event
    event ..> ActionStarted
    event ..> ActionSucceeded
    event ..> ActionFailed
    event ..> WorkflowRejected


    StartWorkflow ..> Workflow

    class  WorkflowService {
        <<service>>
        GetWorkflows(GetWorkflowsRequest) returns (stream GetWorkflowsResponse)
        PublishEvent(PublishEventRequest) returns (PublishEventResponse)

    }

    class GetWorkflowsRequest{
        <<message>>
        string agent_id
    }

    class GetWorkflowsResponse{
        <<message>>
        enum cmd
    }

    class cmd{
        <<Enumeration>>
        StartWorkflow start_workflow
        StopWorkflow stop_workflow
    }

    class StartWorkflow{
        <<message>>
        Workflow workflow 
    }

    class StopWorkflow{
        <<message>>
        string workflow_id 

    }

    class PublishEventRequest{
        <<message>>
        Event event
    }

    class PublishEventResponse{
        <<message>>
    }

    class Workflow{
        <<message>>
        string workflow_id 
        repeated Action actions

    }

    class Action{
        <<message>>
        string id
        string name 
        string image
        optional string cmd
        repeated string args
        map<string, string> env
        repeated string volumes
        optional string network_namespace
    }

    class Event{
        <<message>>
        string workflow_id
        enum event
    }

    class event{
        <<Enumeration>>
        ActionStarted action_started
        ActionSucceeded action_succeeded
        ActionFailed action_failed
        WorkflowRejected workflow_rejected
    }

    class ActionStarted{
        <<message>>
        string action_id
    }

    class ActionSucceeded{
        <<message>>
        string action_id
    }

    class ActionFailed{
        <<message>>
        string action_id
        optional string failure_reason
        optional string failure_message
    }
    
    class WorkflowRejected{
        <<message>>
        string message
    }
```