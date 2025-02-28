# legitimate_events_generator

Mathias:

A new thread - legitmate events generation. In the end i think we want to emulate user behavior and their resulting telemetry events just as another agent. However, as we have discussed we might need specific user graphs for this. Before we have this in place i think we are stuck with generating false positives for the telemetry in the attackgraphs. I think it makes sense to come up with a good format for this as i think we will probsbly have to live with this way of doing it for a while. 16 replies 

Mathias:

My hypothesis would be to specify this in a similar format as we are discussing for mal2.0 above, but in a separate file indicating that it is representing the user agent. So we would copy the telemetry from whatever mallang we are working on and then state probabilities for all the parameters, e.g. ! list_bucket(prob(ip-set), prob(account-set)) [prob] Where we then have listed sets of accounts and ip:s that we have defined as legit. The last probability is the probability that the log is grnerated in every timestep. I think we need to allow somewhat complex probability functions. Perhaps we want to put different probabilities on different combinations of ip and account, change probability over simulation time. We also need to be able to specify this on the instance level. Allowing default values on the class-level that can be overwritten on the instances seems natural to me. (edited) 

Mathias:

I get the feeling that Johan wants to get started pretty soon with an implementation at Google, so something along the lines of this is what i would suggest there.

Pontus:

Regarding the user agents, I'm thinking that it would not be so hard to define these, so I'm leaning towards doing that right away. At the creation of a user agent, an account and an IP is chosen. Assuming we have real logs to base the agents on, we could create an agent for each of the user accounts in the log history, each with the last seen IP address, for instance. At each time step, there would then be two different functions: 
p(change_ip) [as per the frequency in the logs] and 
p(action | time_capped_action_history) [also as per the frequency in the logs] 
The probability that an action is performed is thus dependent on what actions were recently performed. We can make the agent very simple by setting the time_capped_action_history to zero, or it can become more sophisticated by increasing that value. Of course, if we don't have logs, we can just invent the probabilities ourselves. 

Mathias:

Ok, so these are the functions for the list_bucket telemetry event specifically (and for some other events too I assume). But it is not a generic solution to how to specify how to generate legitimate telemetry events. Good! 10:52 Mathias Of course it would be nice to come up with a more formalized way of specifying agent behaviour. But I guess we will run into various things with respect to this, so it is probably good to be a bit iterative here.

Pontus:

@Mathias : My proposal is for a generator of all telemetry events, not only list_bucket. It is intended as a generic solution. 

Mathias:

@Pontus Ok, but then you would need to specify the set of IPs/ things you are allowed to change between. And then we are I guess more or less saying the same thing? Or what is it in my proposal you dont like (if any)? 

Pontus:

I interpreted your proposal as centered on the telemetry events, while I'm proposing that it be centered on the users. For instance, I propose that each user maintains a history of her previous actions. Is that also how you envision it? (edited) 

Mathias:

Yes, this is also how I envision it. And my thinking is that the agent behaviour is specified in a MAL-like syntax when they decide what next attack step to take and what info to "leave" on telemetry events. 

Pontus:

I think a MAL specification of a user’s behavior will be more complicated than what I proposed. While highly desirable, it is not trivial to produce a MAL specification from logs. I think it can be done, but I think it will require some research. My proposal (above) is less ambitious. New 

Mathias:

Fair enough! So what you are saying is that the user agent has every telemetry event represented and for all of them there is a p(action | possibly_something) attached as well as a "latest" value for all employ parameters of the event and a p(change_employ_parameter_value). Good with me.

Nikolaos:

 Given a telemetry event definition as:
! meterpreter_detected(ip) [0.9]
I think the consensus is that the number in the brackets (regardless of whether there will be a second number next to it ever, or not), is the true positive rate:
True Positive Rate = True Positives / (True Positives + False Negatives) I.e. the percent of step executions that produced a log. This is an attribute of the detector and does not depend on the agent behavior.

Nikolaos:

Although, there could be cases where this probability is also conditional on the agent actions. For example, if the user deployed the meterpreter shell in a way that prevents the event from firing. But maybe such things should be taken care of in the attack graph login, for example if the user obfuscates the meterpreter executable to avoid having it be detected, this should be a different attack path that leads to a meterpreter_detected2(ip) [0.1] telemetry event. 

Mathias:

I agree that the probabilities here are true positives, for the respective agent. However, in the legitimates event generation we also need to decide how often the event is triggered. So there is another probability we need to provide there. (Unless we do the generation by a "real" agent simulation.) This was the probability I originally had in mind when starting the discussion. (To me it seems reasonable to state the true positive value in the MAL-spec/simulator/the world once and for all agents. At least for this version of the infrastructure.) 

Pontus:

Agree. 👍

------------

Pontus:

Seems super simple to do self-supervised training with a GNN to predict the next log event. Given the example of LLMs, that ought to imbue the GNN with a deep understanding of the cloud environment. We should be able to produce a "legitimate user graph", a LUG, (as opposed to an attack graph), that specifies the likelihood that a user would perform a certain action at a given point in time. And we probably don't even need such a LUG, as we could directly use the GNN to produce realistic logs.
