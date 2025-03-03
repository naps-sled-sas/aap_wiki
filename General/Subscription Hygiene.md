# THIS DOCUMENT IS IN-PROGRESS

# What is a "Node" and how is it counted?
- The official KB outlining what a node IS can be found [here](https://access.redhat.com/articles/3331481) it is essentially anything AAP contributes to the configuration of. This can be servers, appliances, etc. that it is directly connecting to or downstream devices such as wireless access points if a controller is configured.

- A "seat" is taken after a node is connected to. This means simply importing a server into an inventory does NOT consume a seat.

- So in summary. A "Node" is something AAP has configured not just imported. This includes devices configured by proxy through something like a controller appliance or API endpoint. At the time of writing these downstream devices need to be manually accounted for.

- There is deduplication across inventories but only for matching hostnames. If for example an inventory plugin imports a host using hostname but another inventory is source from a file using IP address. This will result in the same node consuming two seats. It is encouraged to have consistency across your sources or make use of [constructed inventories](https://www.redhat.com/en/blog/how-to-use-the-new-constructed-inventory-feature-in-aap-2.4) to "chop up" a known good source.

# How can I track my subscription?
- Current subscription status is viewable in (Settings -> Subscription)
![image](/Images/sub_hygiene-1.png)

- You can get a more detailed accounting view under (Automation Analytics -> Host Metrics). This data is the source of truth for your consumption.
![image](/Images/sub_hygiene-2.png)

# How can I free up seats if a node no longer exists?
- You can manually remove nodes from the host metrics view but there is also an [API](https://developers.redhat.com/api-catalog/api/ansible-automation-controller) call that can be used (ctrl + f "host metrics"). Example tasks coming soon. 
