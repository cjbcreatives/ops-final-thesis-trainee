# People throw around “multi-master” and “high-availability” a lot. What’s your take?

There’s a difference between high availability and multi-master. If you, for example, have three masters and only one Nginx instance in front load balancing to those masters, you have a multi-master cluster but not a highly available one because your Nginx can go down at any time and well, there you go.