# oci-visualizer
Oracle Cloud Infrstructure  Visualization Tool

# OCI Visualizer

<b>Overview</b>

OCI Visualizer is a tool to visualize an OCI tenancy primarily from a network perspective. The tool was created mainly for the purpose of better understanding the OCI network model. As  a great believer in API first and visualization, I thought that the best way to learn a product is  to combine the two and build a visualization tool. 
I was inspired by <a href="https://duo.com/blog/introducing-cloudmapper-an-aws-visualization-tool">Duo Security CloudMapper.</a> a visualization tool for AWS.  

<b>How it works?</b>
- First visualizer extracts the network data for all VCNs and other cloud resources in an OCI compartment and generates  a json data structure . This is done using a Python script leveraging the OCI  Python SDK and exposed as a flask REST service.

- Visualize the data generated in step 1 using a small web app, leveraging  <a href="http://js.cytoscape.org/">cystoscape.js</a> , an open source graph visualization framework. 

Both the extraction script and the web app run on top of python <a href="http://flask.pocoo.org/">Flask</a>

<p align="center">
  <img src="oci_visuslizer.png" width="800" height="500"/>
</p>



<b> installation </b>

1. <b>Install OCI Python SDK as described <a href="https://oracle-cloud-infrastructure-python-sdk.readthedocs.io/en/latest/">here</a> </b>

	$ yum install  oci-cli
	$ yum install  oci
	
2. <b>Install flask and flask-cors</b>

	$ pip install flask
	
	$ pip install flask-cors

3. <b>Update your OCI configuration file with your tenancy details</b>

	Edit your ~/.oci/config file and setup your tenancy details

4. <b>configure the machine firewall </b> If you’re installing on an OCI Oracle  linux instance 

	$ sudo firewall-cmd --zone=public --add-port=8000/tcp --permanent
	$ sudo firewall-cmd --reload

5. <b>Install the OCI Visualizer tool </b>

	$ tar -xvf  ociviz-0.1.tar.gz
	
	This will create a directory named "ociviz-0.1". 

	$ cd ociviz-0.1
	$ python setup.py develop




6. <b>Launch the flask  web server</b> 

	$ export FLASK_APP=ocivizserver.py 
	$ flask run --port 8000

7. <b>Launch the visualizer UI</b>
	- Point your browser to  http://localhost:8000/static/ociviz.html . 
	- You can use http request parameters to control how Visualizer is launched. the following parameters are available:
	
		- <b>'proxy' </b>- if you're behind a firewall and need to use a proxy:
		
		- <b>'profile'</b> - By default, Visualizer will use the DEFAULT profile on your config file. If you want
		   to use a different  profile then provide a profile parameter in the request url.
		
		- <b>'mode'</b> - 
		
			 <b>'REST'</b>: this is thhe default mode. in this mode the netwrok data is retrieved in real time 
			 by calling the Python SDK.
			 
			 <b>'OFFLINE' </b>: Sometimes a user may not have access the OCI APIs. In such case the tool can 
			   be launched with mode=OFFLINE and will  visualize the content of the file 'network.json' 
			   in the static directory. 
			   
			 <b>'FILE'</b> : to generate a file that can be viewed OFFLINE. The network file will be generated 
			     in the static folder with name "network_<compartmentname>.json"	
			     
		- <b>'compartment'</b> - When Visualizer is launched, it will retrieve all compartments in the tenancy 
		     defined by your profile.
		     if you want to visualize a spcific compartment then use the compartment request parameter.
	
<b>example 1:</b> 

  http://localhost:8000/static/ociviz.html?proxy=myproxy.com:80&profile=myprofile&compartment=mycompartment

will launch visualizer using a proxy at myproxy.com:80 and visualize compartment mycompartment. access to the OCI will be done the myprofile section in the config file at ~/.oci/config

<b>example 2:</b> 

  http://localhost:8000/static/ociviz.html
  
will lunch visualizer without a proxy, will use the DEFAULT profile in the OCI config file and will display a list of compartments for that profile.


<b>example 3:</b> 

  http://localhost:8000/static/ociviz.html?mode=OFFLINE
  
will lunch visualizer  in OFFLINE mode. Visualizer expects to find a file called "network.json" in static folder and will visualize its contents

<b>example 4:</b> 

  http://localhost:8000/static/ociviz.html?mode=FILE
  
will lunch visualizer and will write a file named "network_mycompartment.json' in the static folder, whenever I visualize a compartment 

8. <b>What do I see on the diagram ?</b>
 	- Subnets may be drawn with a gree or red border - Red is for Private subnet while green is for  Public subnets
	- Blue edges repesent routes from a subnet to other subnet/DRG/Local Peering ...
	- Green edges represent ingress security rules for the subnet. Note that egress are not yet implemented
	- Load balancers will have edges pointing to the instances defined in the backend sets. For public load balancer 
	  they will appear in both public subnets , but only one will have edges pointing to the backend set.	

9. <b>What can I do when visualizing a VCN:</b>
	-
	-  Click on a node or edge to display a node or edge properties -
	- Check/unckeck the Routes to show/hide route table based edges
	- Check/unckeck the SecRules  to show/hide edges based on Security rules
	- Check/Unchek the Instances to show/hide instances on the diagram
	- Right click a subnet and dispaly the route table or security rules for that subnet
	- Right click an Instance and display the Attached VNICs or Attached Block Volumes

