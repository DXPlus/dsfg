## DSFG: Data Synchronization for Gossipsub

This project focuses on designing a generic data synchronization mechanism for Gossipsub to achieve fast synchronization of messages within a topic. The code implementation is based on the data synchronization module in Prysm (ETH 2.0 consensus client go language version), and makes it more general and concise by decoupling its logic related to the blockchain. The code repository is not open source yet.

### Overview of Code Design



#### TopicSync

TopicSync is the entry point for data synchronization. It is mainly responsible for calculating the highest synchronization number and the synchronization starting point number, and creating a download queue. TopicSync calls Fetcher to obtain the content hash from other nodes required to calculate the highest synchronization number and the synchronization starting point number. TopicSync requires the upper layer to provide a channel for processing content (chan *queueFetchedData) and an interface function for selecting forks (func selectForks()). TopicSync uses TopicStateManager to obtain the topic state of other nodes.

#### TopicStateManager

TopicStateManager is mainly responsible for initiating requests for topic states of other nodes and responding to requests for topic states from other nodes. TopicStateManager requires the upper layer to provide an interface for accessing its own topic state (func GetCurrentTopicState()). TopicStateManager builds a protocol for transmitting topic state and utilizes libp2p stream to send and receive data.

#### Queque

Queue is responsible for scheduling download tasks, creating and assigning download tasks, and handling various errors that occur during the download process. The Queue calls the FSM Manager to create an FSM to download a batch of content. Multiple FSMs can download content in parallel. The downloaded content will be sent to TopicSync in order for processing. The FSM saves the download state of the batch of content, and the Queue triggers the FSM state transition periodically based on the download state. When an FSM fails to download, the FSM will be put on hold and then reset.

#### FSMManager

FSMManager is responsible for managing FSM, adding and deleting FSM, and calling the processing function of state transfer.

#### FSM

Each state machine FSM is responsible for downloading a fixed amount of content starting from the start  number. The start number is the unique identifier of the FSM. The FSM class saves its own state attributes and the obtained content. The state transition diagram of FSM is as follows:



#### Fetcher

Fetcher is responsible for fetching data from other nodes, including content with specific serial numbers and content hashes. Fetcher builds a protocol for transmitting content and content hashes, and utilizes libp2p's stream to send and receive data. Fetcher requires the upper layer to provide interfaces for obtaining content (func GetContents(topic, from, amount)) and content hashes (func GetContentHashs(topic, from, to)).
