# Dataflow templates 

    Dataflow templates allow you to package a Dataflow pipeline for deployment. 

    Anyone with the correct permissions can then use the template to deploy the packaged pipeline

### Creation of Template:

    1. Creating your Custom Data flow Template 
    
    2. Using pre-built GCP provided Data FLow Template

### Benefits of Template:

1. Templates separate pipeline design from deployment.
   For example, a developer can create a template, and a data scientist can deploy the template at a later time.

2.  Template can have `parameters` that let you `customize` the pipeline when you deploy the template.

3.  Deploy a template by using the Google Cloud console, the Google Cloud CLI, or REST API calls. You don't need a development environment or any pipeline dependencies installed on your local machine.

4.  A template is a `code artifact` that can be `stored` in a `source control repository` and used in `continuous integration` (CI/CD) pipelines.


### Google-provided templates:


-> You Can find variety of pre-built, open source Dataflow templates that you can use for common scenarios. 40 template for both Batch processing and Stream processing

### Compare Flex templates and classic templates (customize template):
-> Dataflow supports two types of template: Flex templates, which are newer, and classic templates. (Flex is Recommended)

    1. Flex templates, the developers package the pipeline into a Docker image, push the image to Artifact Registry, and upload a template specification file to Cloud Storage.

-> The template specification contains a pointer to the Docker image. When you run the template, the Dataflow service starts a launcher VM, pulls the Docker image, and runs the pipeline. 

-> The execution graph is dynamically built based on runtime parameters provided by the user. 

-> To use the API to launch a job that uses a Flex template, use the `projects.locations.flexTemplates.launch` method.
 
    2. classic templates, developers run the pipeline, create a template file, and stage the template to Cloud Storage.

-> A classic template contains the JSON serialization of a Dataflow job graph. 

-> The code for the pipeline must wrap any runtime parameters in the `ValueProvider` interface. This interface allows users to specify `parameter values` when they deploy the template.

-> To use the API to work with classic templates, see the `projects.locations.templates API reference` documentation.

    3. Flex templates have the following advantages over classic templates:
    
    -> Unlike classic templates, Flex templates don't require the ValueProvider interface for input parameters. Not all Dataflow sources and sinks support ValueProvider.
       For example, the template might select a different I/O connector based on input parameters.

    -> While classic templates have a static job graph, Flex templates can dynamically construct the job graph.

    -> A Flex template can perform preprocessing on a virtual machine (VM) during pipeline construction.
      For example, it might validate input parameter values.

# Template workflow

1. Developers set up a development environment and develop their pipeline. The environment includes the Apache Beam SDK and other dependencies.

2. Depending on the template type (Flex or classic):

       For Flex templates, the developers package the pipeline into a Docker image, push the image to Artifact Registry, and upload a template specification file to Cloud Storage.

       For classic templates, developers run the pipeline, create a template file, and stage the template to Cloud Storage.
3. Other users submit a request to the Dataflow service to run the template.

4. Dataflow creates a pipeline from the template. The pipeline can take as much as five to seven minutes to start running.

# Set IAM permissions

### Dataflow jobs, including jobs run from templates, use two IAM service accounts:

1️⃣ Dataflow Service Account → Manages Google Cloud resources like creating VMs.

2️⃣ Worker Service Account → Allows worker VMs to access pipeline files and required resources.

Ensure that these two service accounts have appropriate roles

# Extend templates

-> You can customize Dataflow templates by modifying their code. 
   For example, if a template discards late-arriving data due to a fixed window, you can update it to allow late data using `.withAllowedLateness.`

-> `.withAllowedLateness.` It is a Dataflow pipeline setting that controls how long late-arriving data is accepted in a windowed stream processing job.   
