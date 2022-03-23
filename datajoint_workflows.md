# Workflow Management for DataJoint

**Duration Target:** 14min read (@160 wpm)

**Release:** Thursday, 2022-03-24

**Authors**: Raphael Guzman, Thinh Nguyen, and Dimitri Yatsenko

## Definition 
Let's agree on a few  definitions:

A **job** is a fully-specified request to perform a computation or task in a deterministic manner

Jobs should be specified in their entirety so that they can achieve consistent results in a predictable timeframe. Reliability is achieved if results can be reproduced and independently verifed.

Consider a job that responsible for compiling a `plotly` graph from some input data. For reliable results, we need to answer several questions:

- What version of `plotly` and all its dependencies will we use?
- What is the input data and the processing parameters?
- What hardware resources should we allocate? CPU? GPU? RAM? What operating system and hardware architecture?
- How can the job be parallelized through multithreading and multiprocessing?

These last questions may not affect the final result but they do affect the run time.  Answering these questions leads to the predictability of computation results and execution times.

A **workflow** is the formal specification of the dependency relationship between jobs

Workflows provide instructions for cascading dependent computations or tasks. We can chain jobs together in this way to organize how inputs transform into results through various stages of a workflow. These acyclic relationships are important because they define a specific direction and a finite end to the operation. This is commonly depicted with a directed acyclic graph (DAG).

Continuing with our example, we define two jobs that will consume the generated `plotly` graph: 

- embed the plot within a provided text input (say for a report)
- log the activity to a central service

This workflow has one parent job and two children jobs.

Then **job orchestration** is the process of delegating jobs to both an available and appropriate computing resource.

Think of the job orchestrator layer as a matchmaker of sorts. By listening to a *job queue*, her responsibility is to connect the right job to the right environment; best fit for the job. Notice that the notion of a workflow here really isn't necessary since all we are doing is scheduling and fullfilling computing obligations. This means that as we process the queue, we either need to spawn an environment suitable for the job or wait until there is bandwidth within the landscape (remote or on-premise) to launch it.

Perhaps in our example we'd like for our jobs to be run within a distributed Kubernetes cluster. Jobs could be encapsulated in pods scheduled against nodes that satisfy the hardware requirements.

**Workflow Management** is an interface (graphical or programatic) that enables workflow specification and job queueing

Naturally, users will need to interact with these facilities to perform work. This is usually done through the use of a workflow management tool or service. Their role is to simplify how a user defines workflows and submits them to the job orchestrator. Through the use of a combination of a UI and/or a API, the user typically gains control of:

- when workflows should be submitted
- monitoring and receive notifications of workflow states
- ability to intervene on failed workflows e.g. restarting a workflow

Returing to our example, using a workflow management tool would be how we might define our workflow's jobs and set the workflow to be triggered once a day. Though largely automated now, we'd be wise if during processing we assign someone to be on-call should intervention be necessary.

A **data pipeline** is somet different conceptually.  
It is the formal specification of the data sources, transformations, and all its intermediate structures.
It also takes the form of a directed acyclic graph of dependencies. 
In many cases, the data pipeline provides the basis for the *workflows*.

By now we've defined a nicely determistic system but there is one key issue. Workflows on their own don't appropriately summarize a goal. Generally, they describe a chaining flow of operations or a series of steps within a wider collective effort. As such, they might not clearly represent what the true mission is. Additionally, a workflow expects all inputs to be defined in the same breath to kick-start the process. To help tie things together, it is necessary to consider a data pipeline. A data pipeline strings together multiple workflows to clearly indicate:

- segments where user inputs are necessary
- dependencies between workflows
- representation of the purpose of the effort

To complete our example, our larger effort might be that we are conducting a behavior experiment that demonstrates pose tracking capabilities. We can organize phases of the experiment into dependent workflows:

- workflow: gather video data with some pre-processing, input: source files
- workflow: add pose tracking to videos and update metadata, input: utilize a specific algorithm
- workflow: generate graphs and figures for publication, input: curated list of session data

## Data Pipelines in DataJoint
DataJoint is the framework for a formal specification of data pipelines. 
It does so using the principles of relational databases. 
When relational schemas are restricted to acyclic relationships through foreign key dependencies, then such a schema can serve as both a database and a data pipeline. 
DataJoint tables can specify the computations tied to their referential dependencies. 

- **Data Consistency**: Establishing a single-source-of-truth is a primary requirement within a reseaerch team. 
Proper database systems help avoid data anomalies that arise from concurrent data manipulations by the team.

- **Data integrity**: Understanding the lineage of your data, its origins and relationships is essential. DataJoint uses relational data integrity constraints for ensuring entity integrity, referential integrity, and group integrity in the data pipleines.
Data integrity constraints prevent many errors.

- **Reproducibility**: Making computation a native, natural part of the user's experience with their data model allowed us many benefits. Foreign key relationships facilitate data provenance and referential integrity. This paves the way for automation as we included the feature to attach distributed computation to specific table types. This proves to be a reliable way to determine if automated tables needed to perform new computations. The goal is to ensure reproducibility though I will admit this is *partially* in place since compute environments are not completely defined for full reproducibility.

- **Quickstart**: Nothing is more frustrating that wanting to try a new approach to a compute workflow but getting bogged down by a clunky, convoluted setup. We felt it important to keep things as simple as possible by building client packages in pure Python or MATLAB easily installable on a local machine. 

- **Simplified Queries**: Many of the users we cater to sought for solutions that could help organize and manage the compute needs of their scientific research effort. As an added perk of using DataJoint, their data was now easily accessible for querying though learning SQL was an unfortunate burden for many scientists. Thus, we decided to create our own intuitive DataJoint query syntax that served as a powerful SQL query builder making complex SQL algebra much more accessible to the typical researcher.

- **Interoperability**: Much of the scientific community participates in both the Python and MATLAB ecosystem such that there are excellent open-source projects across them. Choosing to support both communities, we created clients in both enforcing interoperability between them. Meaning that a user could serialize data in one and exchange with the other. This further promotes collaboration through hybrid **data pipelines** that can define some computations to be carried out in Python and some in MATLAB.

## Workflow management in DataJoint 
DataJoint comes with a built-in lightweight workflow management process in its `populate` method. 
This allows coordinating the computations across multiple nodes. 
This process works for projects of medium sizes. 
However, this process lacks many of the capabilities of modern workflow management systems. 


Currently, the DataJoint **data pipeline** operation is scriptable but still largely a manual effort. DataJoint expects the machine where the pipeline is defined to operate as a worker to process jobs. Through `.populate()` we are able to process in bulk jobs within a table but organizing a pipeline-wide populate is largely on the responsibility of the user (currently). Additionally, retrieving logs, debugging, and restarting failed runs is quite involved where DataJoint expects the user to know their way around the infrastructure.

## Minding the Gap

*Let's not reinvent the wheel; instead seek better integration.*

Fostering a healthy community of scientists means we need to allow for individuals to not be faulted for being adventurous and opinionated. There are many amazing tools today that are itching to find a way to help you, and many for a good reason!

Though at DataJoint we largely apply compute management in a Machine Learning and Neuroscience context, in its essence, the problem is one many projects have tackled before. So much so that there is a dense space today of diverse workflow management options. Their nomenclature might be a bit different but in principle they operate similarly and use similar assumptions.

Since we've taken the time to define those common terms above, let's now try to understand how we can enhance the DataJoint open-source framework to be more compatible with workflow management solutions. The key is to see how we can compliment each other.

Let's suppose there are two ways to operate **data pipelines**:

1. Allow DataJoint to consume it's own workflow queue manually treating the client as a worker i.e. as-is with `.populate()`. This is the existing mode of DataJoint and perhaps we might leave this as the default (for now).
1. Through configuration, interface with a larger workflow management system
   - Normalize computation by decoupling from database operations. This will allow us to adhere to principles of proper functional programming.
   - Add capability to perform a callback function as a result of successful inserts. This would be DataJoint's implementation of a `TRIGGER`.
   - Generate independent workflows definitions based on 'staged' data. This means that DataJoint could use its **data pipeline** to create workflows based on data that has been inserted prior to a series of Computed/Imported tables. This 'staged' data would effectively set the boundary conditions to initiate the workflows.
   - Add capability to perform a callback function on workflow generation to redirect the workflow to the target workflow management tool. This callback should respect the workflow dependency order.

Great. The real value here is that DataJoint is helping us automatically generate workflows based on our existing data model's structure. Now that we have an approach to allow coordination with a workflow management solution, let's identify some important requirements of consideration. We may allow flexibility for users to connect their own 'workflow-management-backend', but let's describe some important aspects of a proper workflow management solution that we'd feel comfortable recommeding and using within our production systems.

## [DRAFT] Workflow Management Requirements

- allow for each job to be containerized for environment management and reduced footprint
- scale job performance with minimum overhead from compute algorithm developer
- cache/reuse expensive results if there is no change
- nice authenticated web gui allowing visibility, debugging, notification, traceability, status intervention
- schedule workloads in a dynamically scalable cluster matching the hardware specification of an algorithm with a worker appropriate for it against the appropriate landscape i.e. remote or on-premise cluster
- support an event-driven architecture

## [DRAFT] Workflow Management Solution Review

- prefect
- flyte

## [DRAFT] References & Acknowledgements

## [DRAFT] Outro

- invite into discussion
