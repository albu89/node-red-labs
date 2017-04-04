#  Lab: Watson Visual Recognition with Node-RED
## Overview
The Watson  Visual Recognition service allows to analyse the contents of an image and produce a series of text classifiers with a confidence index.

## Node-RED Watson Visual Recognition node
The Node-RED ![`VisualRecognition`](images/node_red_watson_visual_recognition.png) node provides a very easy wrapper node that takes an image URL or binary stream as input, and produces a set of image labels as output.

## Watson Visual Recognition Flow construction
In this exercise, we will show how to simply generate the labels from an image URL.

### Prerequisites and setup
To get the service credentials automatically filled-in by Node-RED from the BlueMix credentials, the Visual Recognition Service should be bound to the Node-RED application in BlueMix.

![](images/reco_lab_visual_recognition_service.png)

Please refer to the [Node-RED setup lab](/introduction_to_node_red/README.md) for instructions.

### Building the flow
The flow will present a simple Web page with a text field where to input the image's URL, then submit it to Watson Visual Recognition, and output the labels that have been found on the reply Web page.
![Reco-Lab-VisualRecognitionFlow.png](images/reco_lab_visual_recognition_flow.png)
The nodes required to build this flow are:

 - A ![`HTTPInput`](/introduction_to_node_red/images/node_red_httpinput.png) node, configured with a `/reco` URL
 - A ![`switch`](/introduction_to_node_red/images/node_red_switch.png) node which will test for the presence of the `imageurl` query parameter:
   ![Reco-Lab-Switch-Node-Props](images/reco_lab_switch_node_props.png)
 - A first ![template](/introduction_to_node_red/images/node_red_template.png) node, configured to output an HTML input field and suggest a few selected images taken from the main Watson Visual Recognition demo web page:
```HTML
    <h1>Welcome to the Watson Visual Recognition Demo on Node-RED</h1>
    <h2>Select an image URL</h2>
    <form  action="{{req._parsedUrl.pathname}}">
        <img src="http://img.usmagazine.com/article-leads-vertical-300/1250531162_britney_spears_290x402.jpg" height='100'/>
        <img src="http://www.atpworldtour.com/~/media/tennis/players/head-shot/vibrant/federer-headshot15.png" height='100'/>
        <img src="http://cdn1.spiegel.de/images/image-1125664-860_poster_16x9-wkbp-1125664.jpg" height='100'/>
        <br/>Copy above image location URL or enter any image URL:<br/>
        <input type="text" name="imageurl"/>
        <input type="submit" value="Analyze"/>
    </form>
```
![Reco-Lab-Template1-Node-Props](images/reco_lab_template1_node_props.png)
 
- A ![change](/introduction_to_node_red/images/node_red_change.png) node (named `Extract img URL` here) to extract the `imageurl` query parameter from the web request and assign it to the payload to be provided as input to the Visual Recognition node:
![Reco-Lab-Change_and_Reco-Node-Props](images/reco_lab_change_and_reco_node_props.png)

 - The ![Watson Visual Recognition](images/node_red_watson_visual_recognition.png) node. Make sure that the credentials are setup from bluemix, i.e. that the service is bound to the application. This can be verified by checking that the properties for the Visual Recognition node are clear. Furthermore ensure that the node is set to detect faces.
 ![Visual Recognition node properties](images/reco_lab_visual_recognition_service_credentials.png)

 - And a final  ![`template`](/introduction_to_node_red/images/node_red_template.png) node linked to the ![`HTTPResponse`](/introduction_to_node_red/images/node_red_httpresponse.png) output node. The template will format the output returned from the Visual Recognition node into an HTML table for easier reading:
```HTML
<h1>Visual Recognition v3 Image Analysis</h1>
    <p>Analyzed image: {{result.images.0.resolved_url}}<br/><img id="image" src="{{result.images.0.resolved_url}}" height="200"/></p>
    {{^result}}
        <P>No Face detected</P>
    {{/result}}
    <p>Images Processed: {{result.images_processed}}</p>
    <table border='1'>
        <thead><tr><th>Age Range</th><th>Confidence</th><th>Gender</th><th>Confidence</th><th>Name</th></tr></thead>
        {{#result.images.0.faces}}<tr>
            <td><b>{{age.min}} - {{age.max}}</b></td><td><i>{{age.score}}</i></td>
            <td>{{gender.gender}}</td><td>{{gender.score}}</td>
            <td>{{identity.name}} ({{identity.score}})</td>
        </tr>{{/result.images.0.faces}}
    </table>
    <form  action="{{req._parsedUrl.pathname}}">
        <br><input type="submit" value="Try again or go back to the home page"/>
    </form>
```
![Reco-Lab-TemplateReport-Node-Props](images/reco_lab_templatereport_node_props.png)  
Note that the HTML snippet above has been simplified and stripped out of non-essential HTML tags, the completed flow solution has a complete HTML page.

### Testing the flow
To run the web page, point your browser to  `/http://xxxx.mybluemix.net/reco` and enter the URL of some  image.
The URL of the pre-selected images can be copied to clipboard and pasted into the text field.

The Watson Visual Recognition API will return an array with the recognized features, which will be formatted in a HTML table by the template:


### Flow source
The complete flow is available at [Reco-Lab-WebPage](reco_lab_web_page.json).
