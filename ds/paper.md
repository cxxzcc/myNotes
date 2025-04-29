

classification
Data Management and Consistency
* Shared Database : no data consistency issues
* Database per Service
	* Two Phase Commit Protocol ：not scale
	* Saga approach : which supports eventual consistency through the orchestration, or choreography, of several local transactions that can rollback using com pensating transactions

Communication Style:
* Synchronous Communication
* Asynchronous Communication

Service Orchestration: 
* Bare Machine Based
* Virtual Machine Based
* Container Based