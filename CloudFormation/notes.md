# CloudFormation

## Intrinsic Functions
* Ref & Fn::GetAtt
	* using !Ref on template or pseudo params returns their value. When used with logical resources - the physical ID is usually returned
	* GetAtt can be used to retrieve any attribute associated with the resource. Most logical resources return detailed configuration of the physical resource

* Fn::Join & Fn::Split for string manipulation

* Fn:GetAZs & Fn::Select
	* GetAZs is an environmental awareness function. Returns a list of AZs in the explicit region, or current region. 
	* Returns a list where subnets exist in the Default VPC that is being used
	* SELECT, or !Select, returns an object from a list starting at index 0

* Conditions (Fn:: IF, And, Equals, Not & OR)

* Fn:Base64 & Fn::Sub
	* 'UserData:' requires base64 encoided data which is passed to the instance
	* Used typically to write automation scripts for EC2 instances
	* Sub is used for find and replace inside the Base64 functionality

* Fn::Cidr
	* You have to provide a CIDR range for VPCs
	* Used to generate a number of smaller CIDR ranges for subnets, from a larger VPC range

## Mappings
Mappings are used to IMPROVE template portability
* Maps keys to values-- allowing lookup
* Can have top and second level keys (think AMI keys for region/architecture)
* !FindInMap -- common use case for top level lookups (finding ami key/val pairs for region/arch)

## Outputs
Using outputs is completely optional, however they are required when resources need to be accessible
from a **parent stack** when using nesting. Can be **exported** allowed cross-stack referencing.

## Conditions
* Declared in an optional 'Conditions' section and is used in the 'Condition' field
* Processed BEFORE resources are created

## DependsOn
CF tries to do things efficiently in parallel using a **Dependency Order**. DependsOn lets you explicitly define this ordering.
* A good use case for this is creating an Elastic IP.
* An Elastic IP requires an IGW attached to a VPC in order to work, so we need to explicitly DependOn the IGW
* DependsOn can either be a single resource or a list of resources

## WaitConditions, Creation Policy and cfn-signal
cfn-signal configures cloudformation to **hold** for X number of **success** signals; wait for **timeout** for those sigals (H:M:S up to 12 hours maximum); amd, if success signals received we
get a *CREATE_COMPLETE*. It can send failure signals though -- timeouts are implicit failures.
* Logical resources are things that are being signaled

CreationPolicy applies a signal requirement (COUNT) and timeout (PT..). 
* Typically used for EC2 instances or Auto Scaling Groups...

WaitCondition's operate similarly to CreationPolicy's. 
* can depend on other resources
* other resources can depend on the WaitCondition
* JSON data passed back in signal respponse can be accessed using a !GetAtt WaitCondition.Data
	* This is useful for licencing with WaitConditions

## Nested Stacks
Resources in a single stack **share a lifecycle**
* there is a limit of 500 resources per stack
* cannot easily reuse resources
	* by design are isolated by default
	* can't reference this in other stacks

Architecture consists of a **ROOT** stack and a **PARENT** stack
* Root stack gets created by a user
	* Root stack is just a normal stack -- nothing special about them
* Parent stack is something that has its own nested stack
* All nested stacks need to be marked as complete before the root stack is marked as complete

Things to consider:
* Whole templates can be reused in other stacks
	* reuse the template - NOT THE RESOURCES
* Use when the stacks **form part of on solution - lifecycle linked**
* Use when you want to overcome the 500 resource limit
	* each nested stack as a separate 500 resource limit


## Cross-Stack References
CFN Stacks are designed to be ISOLATED and SELF-CONTAINED. This is why we have cross-stack references.
* Outputs can be **exported** making them visible from other stacks
* Exports must have a **unique name** in the region
	* Use `Fn::ImportValue ResourceName` insted of !Ref
`	* Need to use the `Export` keyword

When to use
* When you are creating **Service-Orientated** resources with different lifecylces
	* i.e. want to share a VPC
* When you want to re-use resources


## StackSets
Allow you to use cloudformation to deploy across many accounts & regions.
* Think of stacksets as **containers** in an **admin account**
* contain stack instances which reference stacks (1 region in 1 AWS account)
	* Remains even due to failures
* Created in target accounts (an AWS account)
* Security = **self-managed** or **service-managed**
* Terms:
	* Concurrent Accounts
		* Throttles deployments to only deploy CONCURRENT stacks at a time (i.e. conc=2 for 10 deployments does 5 batches)
	* Failure tolerance
		* Amount of deployments that can fail before the stackset fails
	* Retain stacks
		* Removes stack instances (but can retain them)

## DelectionPolicy
If you **delete** a logical resource from a template, then by default, the phhysical resource is deleted.
To overcome this, you can use a deletion policy using:
* DELETE (default);
* RETAIN; or,
* SNAPSHOT (if supported).
	* EBS Volumne, ElastiCache, Neptune, RDS, Redshift
	* Snapshots continue past the Stack lifetime ($$)
* ONLY APPLIES TO DELETE -- NOT REPLACE
	* DATA IS LOST ON REPLACE

## Stack Roles
* CFN uses the **permissions** of the **logged in identity**
* Which means you need permissions for AWS
* CFN can assume a role to gain the permissions
* The identity creating the stack doesn't need resource permissions - only **PassRole**

## CloudFormation::Init
Configuration management system, runs once during bootstrapping userdata on EC2, stored in a CF template
* AWS::CloudFormation::Init part of an EC2 instance logical resource
* Procedural - HOW (User-Data)
* Desired Date - WHAT (cfin-init)
	* if something is already true, nothing happens

## cfn-hup
* If CloudFormation::Init is updated **it isn't rerun**
* cfn-up helper is a daemon which canbe installed and detects changes in resource metadata
* Update stack -> update ec2

## ChangeSets
This is analagous to Terraform **PLAN**! The usual CREATE/DELETE/CHANGE display.

## Custom Resources
CloudFormation does not support everything in AWS. This is like Terraform NULL RESOURCES
* also allows integration with external systems
* Normally use thsi to upload items to s3 after creation, or delete s3 items before deletion (otherwise error)
* Can privison non-aws resources (cool)
* Sends data to an endpoint (lambda/sns) and gets data back from something (lambda is invoked and provided with event information)
	* Sends a success response to a response URL


