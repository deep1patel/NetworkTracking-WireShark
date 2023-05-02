# NetworkTracking-WireShark-GoogleMaps-Python
Network Traffic visualization using the Python programming language, Wireshark and Google Maps.

This project involved developing a Python-based cybersecurity tool that utilizes Wireshark to track network traffic and then visualizes that data on Google Maps. It could potentially help organizations identify and address vulnerabilities in their networks.

Downloading external material
Besides the Python code that will be created in this tutorial, several external resources and applications is needed to make it work.

GeoLiteCity
First of you will need to download the GeoLiteCity database as this will be used to translate a IP address into a Geo location(longitude & latitude). The database can be downloaded here: https://github.com/mbcc2006/GeoLiteCity-data

Wireshark
To capture network traffic for the Python script and display it on Google Maps, you will require the Wireshark application in addition to the GeoLiteCity database. The Wireshark application can be downloaded from https://www.wireshark.org/.

Python IDE - Pycharm(Latest Version)

Capture Network Traffic
Once Wireshark has been installed, the next step is to generate the input data, which will comprise a pcap file capturing all network traffic to and from the device during the capture period. To initiate the capture, launch Wireshark and choose an interface that is receiving traffic, such as:

![image](https://user-images.githubusercontent.com/54338389/235779415-9daf4805-4447-4477-9fea-ea5d51b92fab.png)

When selecting a interface, Wireshark automatically starts a new capture, which is why you immediately gets prompted with network traffic. To stop the capture and save the data you will have to do the following:

![image](https://user-images.githubusercontent.com/54338389/235779691-6126b9ab-75ee-4133-82b1-560f2e6a3ae3.png)
Stop Capture (Red Square)

Once the capture has been stopped you need to export the captured data in pcap format, this can be done by clicking File -> Export Specified Packets.. and then selecting the following format:

![image](https://user-images.githubusercontent.com/54338389/235779728-1db07e2c-f0ec-4638-a9e4-43c077805ebb.png)
pcap format

With the captured data saved in the correct format, you’re ready to move into the Python Implementation.


Python Implementation
First off you will need to import the needed libraries, these are as follows:

import dpkt
import socket
import pygeoip

gi = pygeoip.GeoIP('GeoLiteCity.dat')
As you will notice a variable is declared with the GeoLiteCity database that we downloaded earlier. Be sure to put it in the root folder of your Python project if you’re not to change the path of the variable.


Files in root folder
Next off you will be implementing the main method, which will open your captured data along with creating the header and footer of the KML file, which is the output file that we will upload to Google Maps. Again be sure to place you captured .pcap file in the root of the project and change the name of the code below if you did not save it under the name wire.pcap

def main():
    f = open('wire.pcap', 'rb')
    pcap = dpkt.pcap.Reader(f)
    kmlheader = '<?xml version="1.0" encoding="UTF-8"?> \n<kml xmlns="http://www.opengis.net/kml/2.2">\n<Document>\n'\
    '<Style id="transBluePoly">' \
                '<LineStyle>' \
                '<width>1.5</width>' \
                '<color>501400E6</color>' \
                '</LineStyle>' \
                '</Style>'
    kmlfooter = '</Document>\n</kml>\n'
    kmldoc=kmlheader+plotIPs(pcap)+kmlfooter
    print(kmldoc)
    
If you wish to change the styling of the lines that will be drawn in Google Maps then it’s in the above code that you will be doing so under the tags <style></style>. In this tutorial I have just adjusted the line width and color but other style elements can be added like icons etc. (Just do a quick google search on how to create KML files for Google Maps)

Next of we’re to add the method that will loop over our captured network data and extract the IP’s.

def plotIPs(pcap):
    kmlPts = ''
    for (ts, buf) in pcap:
        try:
            eth = dpkt.ethernet.Ethernet(buf)
            ip = eth.data
            src = socket.inet_ntoa(ip.src)
            dst = socket.inet_ntoa(ip.dst)
            KML = retKML(dst, src)
            kmlPts = kmlPts + KML
        except:
            pass
    return kmlPts
In the above method plotIPs(pcap) the application will loop over our pcap data and extract source and destination IP adresses of each captured network packet.

But our IP’s can’t be used alone as input to Google Maps, first we need to attach a Geo location. This will be done using the following method:

def retKML(dstip, srcip):
    dst = gi.record_by_name(dstip)
    src = gi.record_by_name('x.xxx.xxx.xxx')
    try:
        dstlongitude = dst['longitude']
        dstlatitude = dst['latitude']
        srclongitude = src['longitude']
        srclatitude = src['latitude']
        kml = (
            '<Placemark>\n'
            '<name>%s</name>\n'
            '<extrude>1</extrude>\n'
            '<tessellate>1</tessellate>\n'
            '<styleUrl>#transBluePoly</styleUrl>\n'
            '<LineString>\n'
            '<coordinates>%6f,%6f\n%6f,%6f</coordinates>\n'
            '</LineString>\n'
            '</Placemark>\n'
        )%(dstip, dstlongitude, dstlatitude, srclongitude, srclatitude)
        return kml
    except:
        return ''
In the above method Geo locations is located for each IP address followed by a convertion into KML format. Be aware that in the above example I have hidden my own source IP which is why it says x.xxx.xxx.xxx, for yours to work you can just navigate to https://www.whatsmyip.org/ and locate your own and change it in the code. The reason why you need to input this is because our captured pcap data file has saved our internal IP which is not known outside our own home network. Feel free to extend the code to capture the external IP automatically. This however is out of scope for this tutorial.

When the above code is executed using:

if __name__ == '__main__':
    main()
You will get a complete KML file printed to the terminal. What is left to do is to copy this data into a file and give it a .kml extension.

Adding Data to Google Maps
With our newly created .kml file, we’re now ready to add it to Google Maps. Navigate to https://www.google.com/mymaps. Here you can create a new map and Import a new layer.

![image](https://user-images.githubusercontent.com/54338389/235779997-6b362ce1-f0cc-402e-bcbf-f19f1ac5e2e3.png)
Import new Layer

Click on Import and then select the .kml file on your computer, once the file is uploaded you should see the network traffic being dislayed on the map. The example below is based on my captured data.

![image](https://user-images.githubusercontent.com/54338389/235780094-c6cfc587-9007-470d-8760-dadea9fe6197.png)


