---
id: image-processing
title: Image and Video Processing
---

### Background

Nowadays, the sizes of image and video collections are increasing dramatically along with the need for the computational capacity to process them. 
Even though hardware is getting better and better each day, the requirements for image and video processing applications scale alongside them with each feature added. Large volumes cannot be analyzed on single computer within reasonable time and it highlights the need for distributing the workload among many PCs. Common workloads that would benefit parallelisation include:

* Compression
* Resizing
* Conversion
* Rendering
* Classification

### How can PowerTree help?

Manipulating a homogenous collection of items distributedly in PowerTree is easy and straightforward.
Workflows are always executed across available compute nodes in the cluster and distributing them is as easy as dividing up the work.
For images, it might be as simple as starting a new workflow for each of them.

PowerTree also allows building complex transformation pipelines with relative ease through its simple programming model.
It allows you to express complex processes without concerns about networks, durability and availability.

### Example
<!--DOCUSAURUS_CODE_TABS-->
<!-- Parent Workflow -->

```c#
public class CompressImagesWorkflow : Workflow
{
    [DataMember]
    private bool hasSubworkflows;

    [DataMember]
    public string[] ImageIds { get; set; }

    public override Continuation DoWork(WorkflowRuntime runtime)
    {
        if (!this.hasSubworkflows) 
        {
            foreach (string imageId in this.ImageIds)
            {
                var subworkflow = new CompressSingleWorkflow()
                {
                    ImageId = imageId
                };

                runtime.Subworkflows.Enqueue(subworkflow, timeout: TimeSpan.FromMinutes(2));
            }

            this.hasSubworkflows = true;
        }
        else
        {
            this.NotifyCompleted(runtime.Subworkflows);
        }
        
        return Continuation.WaitAll();
    }

    ...
}
```

<!-- Subworkflow -->
```c#
public class CompressSingleWorkflow : Workflow
{
    [DataMember]
    public string ImageId { get; set; }

    public override Continuation DoWorkAsync(WorkflowRuntime runtime)
    {
        var image = this.GetImage(this.ImageId);
        var compressed = this.Compress(image);
        this.Persist(compressed, this.ImageId);

        return Continuation.Default();
    }

    ...
}
```


<!--END_DOCUSAURUS_CODE_TABS-->
