
# Earthquake data analysis & visualization using Python

##### This notebook provides a quick overview and will help you understand some of the basic functionalities of Python.

*We will be using some of the popular libraries like Pandas, Requests, Folium, JSON etc.*

#### Activities we gonna perform
1.      Data collection : [US Geological Service](https://earthquake.usgs.gov/data/data.php#eq) host different publically available and free to use Rest API to expose data about historical (and live) earthquakes in US.
2.      We will start with creating a request that returns all earthquake data for Jan 2016 in geojson format
3.      We will utilise some basic OOPs concepts to create a suitable class definition to fetch Earthquake data.
4.      We will call this HTTP API request from Python and store the results in different Python data structure.
5.      We will refine and process the raw GeoJson data, and store them in Pandas DataFrame.
6.      We will extend the program to fetch all earthquake data for the year 2016 – _This cannot be done in a single call, as there is a limit of 20,000 events per request. We will use **parallel request implementations** to speed up data gathering._
7.      Use a suitable Python visualisation library e.g. matplotlib, to visualise the data. Use your creativity and see whether there are any conclusions you can draw from the data
8.  Look to create unit test/s and suitable integration test/s to aid your codebase.
9. Techniques used to make the code of production quality.  Creativity in visualisation.  Pandas Dataframes.


##### GeoJSON Summary Format

Description : GeoJSON is a format for encoding a variety of geographic data structures. A GeoJSON object may represent a geometry, a feature, or a collection of features. GeoJSON uses the JSON standard. 
See the GeoJSON site for more information.
    
Usage: GeoJSON is intended to be used as a programmatic interface for applications.

Output
```{
type: "FeatureCollection",
metadata: {
generated: Long Integer,
    url: String,
    title: String,
    api: String,
    count: Integer,
    status: Integer
  },
  bbox: [
    minimum longitude,
    minimum latitude,
    minimum depth,
    maximum longitude,
    maximum latitude,
    maximum depth
  ],
  features: [
    {
      type: "Feature",
      properties: {
        mag: Decimal,
        place: String,
        time: Long Integer,
        updated: Long Integer,
        tz: Integer,
        url: String,
        detail: String,
        felt:Integer,
        cdi: Decimal,
        mmi: Decimal,
        alert: String,
        status: String,
        tsunami: Integer,
        sig:Integer,
        net: String,
        code: String,
        ids: String,
        sources: String,
        types: String,
        nst: Integer,
        dmin: Decimal,
        rms: Decimal,
        gap: Decimal,
        magType: String,
        type: String
      },
      geometry: {
        type: "Point",
        coordinates: [
          longitude,
          latitude,
          depth
        ]
      },
      id: String
    },
    …
  ]
  }```

### Data Dictonary
Below is the list of all the variables that we get from the USGS' APIs.

** alert **
	- Data Type : String
	- Typical Values : “green”, “yellow”, “orange”, “red”.
	- Description :The alert level from the PAGER earthquake impact scale.



** cdi **
	- Data Type : Decimal
	- Typical Values : [0.0, 10.0]
	- Description :The maximum reported intensity for the event. Computed by DYFI. While typically reported as a roman numeral, for the purposes of this API, intensity is expected as the decimal equivalent of the roman numeral. Learn more about magnitude vs. intensity.



** code **
	- Data Type : String
	- Typical Values : "2013lgaz", "c000f1jy", "71935551"
	- Description :An identifying code assigned by - and unique from - the corresponding source for the event.



** depth **
	- Data Type : Decimal
	- Typical Values : [0, 1000]
	- Description :Depth of the event in kilometers.
	- Additional Information : The depth where the earthquake begins to rupture. This depth may be relative to the WGS84 geoid, mean sea-level, or the average elevation of the seismic stations which provided arrival-time data for the earthquake location. The choice of reference depth is dependent on the method used to locate the earthquake, which varies by seismic network. Since ComCat includes data from many different seismic networks, the process for determining the depth is different for different events. The depth is the least-constrained parameter in the earthquake location, and the error bars are generally larger than the variation due to different depth determination methods. Sometimes when depth is poorly constrained by available seismic data, the location program will set the depth at a fixed value. For example, 33 km is often used as a default depth for earthquakes determined to be shallow, but whose depth is not satisfactorily determined by the data, whereas default depths of 5 or 10 km are often used in mid-continental areas and on mid-ocean ridges since earthquakes in these areas are usually shallower than 33 km.



** depthError **
	- Data Type : Decimal
	- Typical Values : [0, 100]
	- Description :Uncertainty of reported depth of the event in kilometers.
	- Additional Information : The depth error, in km, defined as the largest projection of the three principal errors on a vertical line.



** detail **
	- Data Type : String
	- - Description : Link to GeoJSON detail feed from a GeoJSON summary feed.
	NOTE: When searching and using geojson with callback, no callback is included in the detail url. :


** dmin **
	- Data Type : Decimal
	- Typical Values : [0.4, 7.1]
	- Description :Horizontal distance from the epicenter to the nearest station (in degrees). 1 degree is approximately 111.2 kilometers. In general, the smaller this number, the more reliable is the calculated depth of the earthquake.



** felt **
	- Data Type : Integer
	- Typical Values : [44, 843]
	- Description :The total number of felt reports submitted to the DYFI? system.



** gap **
	- Data Type : Decimal
	- Typical Values : [0.0, 180.0]
	- Description :The largest azimuthal gap between azimuthally adjacent stations (in degrees). In general, the smaller this number, the more reliable is the calculated horizontal position of the earthquake. Earthquake locations in which the azimuthal gap exceeds 180 degrees typically have large location and depth uncertainties.



** horizontalError **
	- Data Type : Decimal
	- Typical Values : [0, 100]
	- Description :Uncertainty of reported location of the event in kilometers.
	- Additional Information : The horizontal location error, in km, defined as the length of the largest projection of the three principal errors on a horizontal plane. The principal errors are the major axes of the error ellipsoid, and are mutually perpendicular. The horizontal and vertical uncertainties in an event's location varies from about 100 m horizontally and 300 meters vertically for the best located events, those in the middle of densely spaced seismograph networks, to 10s of kilometers for global events in many parts of the world. We report an "unknown" value if the contributing seismic network does not supply uncertainty estimates.



** id **
	- Data Type : String
	- Typical Values : A (generally) two-character network identifier with a (generally) eight-character network-assigned code.
	- Description :A unique identifier for the event. This is the current preferred id for the event, and may change over time. See the "ids" GeoJSON format property.



** ids **
	- Data Type : String
	- Typical Values : ",ci15296281,us2013mqbd,at00mji9pf,"
	- Description :A comma-separated list of event ids that are associated to an event.



** latitude **
	- Data Type : Decimal
	- Typical Values : [-90.0, 90.0]
	- Description :Decimal degrees latitude. Negative values for southern latitudes.
	- Additional Information : An earthquake begins to rupture at a hypocenter which is defined by a position on the surface of the earth (epicenter) and a depth below this point (focal depth). We provide the coordinates of the epicenter in units of latitude and longitude. The latitude is the number of degrees north (N) or south (S) of the equator and varies from 0 at the equator to 90 at the poles. The longitude is the number of degrees east (E) or west (W) of the prime meridian which runs through Greenwich, England. The longitude varies from 0 at Greenwich to 180 and the E or W shows the direction from Greenwich. Coordinates are given in the WGS84 reference frame. The position uncertainty of the hypocenter location varies from about 100 m horizontally and 300 meters vertically for the best located events, those in the middle of densely spaced seismograph networks, to 10s of kilometers for global events in many parts of the world.



** locationSource **
	- Data Type : String
	- Typical Values : ak, at, ci, hv, ld, mb, nc, nm, nn, pr, pt, se, us, uu, uw
	- Description :The network that originally authored the reported location of this event.



** longitude **
	- Data Type : Decimal
	- Typical Values : [-180.0, 180.0]
	- Description :Decimal degrees longitude. Negative values for western longitudes.
	- Additional Information : An earthquake begins to rupture at a hypocenter which is defined by a position on the surface of the earth (epicenter) and a depth below this point (focal depth). We provide the coordinates of the epicenter in units of latitude and longitude. The latitude is the number of degrees north (N) or south (S) of the equator and varies from 0 at the equator to 90 at the poles. The longitude is the number of degrees east (E) or west (W) of the prime meridian which runs through Greenwich, England. The longitude varies from 0 at Greenwich to 180 and the E or W shows the direction from Greenwich. Coordinates are given in the WGS84 reference frame. The position uncertainty of the hypocenter location varies from about 100 m horizontally and 300 meters vertically for the best located events, those in the middle of densely spaced seismograph networks, to 10s of kilometers for global events in many parts of the world.



** mag **
	- Data Type : Decimal
	- Typical Values : [-1.0, 10.0]
	- Description :The magnitude for the event. See also magType.
	- Additional Information : The magnitude reported is that which the U.S. Geological Survey considers official for this earthquake, and was the best available estimate of the earthquake’s size, at the time that this page was created. Other magnitudes associated with web pages linked from here are those determined at various times following the earthquake with different types of seismic data. Although they are legitimate estimates of magnitude, the U.S. Geological Survey does not consider them to be the preferred "official" magnitude for the event. Earthquake magnitude is a measure of the size of an earthquake at its source. It is a logarithmic measure. At the same distance from the earthquake, the amplitude of the seismic waves from which the magnitude is determined are approximately 10 times as large during a magnitude 5 earthquake as during a magnitude 4 earthquake. The total amount of energy released by the earthquake usually goes up by a larger factor: for many commonly used magnitude types, the total energy of an average earthquake goes up by a factor of approximately 32 for each unit increase in magnitude. There are various ways that magnitude may be calculated from seismograms. Different methods are effective for different sizes of earthquakes and different distances between the earthquake source and the recording station. The various magnitude types are generally defined so as to yield magnitude values that agree to within a few-tenths of a magnitude-unit for earthquakes in a middle range of recorded-earthquake sizes, but the various magnitude-types may have values that differ by more than a magnitude-unit for very large and very small earthquakes as well as for some specific classes of seismic source. This is because earthquakes are commonly complex events that release energy over a wide range of frequencies and at varying amounts as the faulting or rupture process occurs. The various types of magnitude measure different aspects of the seismic radiation (e.g., low-frequency energy vs. high-frequency energy). The relationship among values of different magnitude types that are assigned to a particular seismic event may enable the seismologist to better understand the processes at the focus of the seismic event. The various magnitude-types are not all available at the same time for a particular earthquake. Preliminary magnitudes based on incomplete but rapidly-available data are sometimes estimated and reported. For example, the Tsunami Warning Centers will calculate a preliminary magnitude and location for an event as soon as sufficient data are available to make an estimate. In this case, time is of the essence in order to broadcast a warning if tsunami waves are likely to be generated by the event. Such preliminary magnitudes are superseded by improved estimates of magnitude as more data become available. For large earthquakes of the present era, the magnitude that is ultimately selected as the preferred magnitude for reporting to the public is commonly a moment magnitude that is based on the scalar seismic-moment of an earthquake determined by calculation of the seismic moment-tensor that best accounts for the character of the seismic waves generated by the earthquake. The scalar seismic-moment, a parameter of the seismic moment-tensor, can also be estimated via the multiplicative product rigidity of faulted rock x area of fault rupture x average fault displacement during the earthquake.



** magError **
	- Data Type : Decimal
	- Typical Values : [0, 100]
	- Description :Uncertainty of reported magnitude of the event. The estimated standard error of the magnitude. The uncertainty corresponds to the specific magnitude type being reported and does not take into account magnitude variations and biases between different magnitude scales. We report an "unknown" value if the contributing seismic network does not supply uncertainty estimates.



** magNst **
	- Data Type : Integer
	- - Description : The total number of seismic stations used to calculate the magnitude for this earthquake.
	 :

** magSource **
	- Data Type : String
	- Typical Values : ak, at, ci, hv, ld, mb, nc, nm, nn, pr, pt, se, us, uu, uw
	- Description :Network that originally authored the reported magnitude for this event.



** magType **
	- Data Type : String
	- Typical Values : “Md”, “Ml”, “Ms”, “Mw”, “Me”, “Mi”, “Mb”, “MLg”
	- Description :The method or algorithm used to calculate the preferred magnitude for the event.
	- Additional Information : 

| Magnitude type | Magnitude Range | Distance Range | Comments |
| ----------- | ----------- |----------- | ----------- | ----------- |
| Duration (Md or md) | < 4 | 0 - 400 km | Based on the duration of shaking as measured by the time decay of the amplitude of the seismogram. Often used to compute magnitude from seismograms with “clipped” waveforms due to limited dynamic recording range of analog instrumentation, which makes it impossible to measure peak amplitudes. |
| Local (ML Ml, or ml) | 2 - 7.5 | 0 - 600 km | The original magnitude relationship defined by Richter and Gutenberg for local  earthquakes in 1935. It is based on the maximum amplitude of a seismogram recorded on a Wood-Anderson torsion seismograph. Although  these instruments are no longer widely in use, ML values are calculated using modern instrumentation with appropriate adjustments.
| Short-period surface wave (mb_Lg, mb_lg, or MLg) | 3.5 - 7 | 150 – 1100 km | A magnitude for regional earthquakes based on the  amplitude of the Lg surface waves as recorded on short-period instruments. |
| Short-period body wave (mb) | 43285 | 15 - 100 degrees | Based on the amplitude of P body-waves as recorded on short-period instruments that are most sensitive to waves with a period of about 1 s. |
| Twenty-second surface wave (Ms or Ms_20) | 5 - 8.5 | 20 - 160 degrees | A magnitude for distant earthquakes based on the amplitude of Rayleigh surface waves measured at a period near 20 sec. |
| Moment (generic notation Mw or mw. Specific types denoted Mwb or mwb, Mwc or mwc, Mwr or mwr, and Mww or mww) | > 3.5 | all | Based on the scalar seismic-moment of the earthquake, as determined by a moment-tensor inversion. Mwb – Mw based on moment tensor inversion of long-period (~10 - 100 s) body-waves (P- and SH). Mwc -- Moment magnitude derived from a centroid moment tensor inversion of intermediate- and long-period body- and surface-waves. Mwr -- Moment magnitude derived from a moment tensor inversion of complete waveforms at regional distances (less than ~13 degrees). Sometimes called RMT. Mww -- Moment magnitude derived from a centroid moment tensor inversion of the W-phase. |
| Moment (Mi or Mwp) | 43317 | all | Based on an estimate of moment calculated from the integral of the displacement of the P wave recorded on broadband instruments. |
| Energy (Me) | > 3.5 | all | Based on the seismic energy radiated by the earthquake as estimated by integration of digital waveforms |


** mmi **
	- Data Type : Decimal
	- Typical Values : [0.0, 10.0]
	- Description :The maximum estimated instrumental intensity for the event. Computed by ShakeMap. While typically reported as a roman numeral, for the purposes of this API, intensity is expected as the decimal equivalent of the roman numeral. Learn more about magnitude vs. intensity.



** net **
	- Data Type : String
	- Typical Values : ak, at, ci, hv, ld, mb, nc, nm, nn, pr, pt, se, us, uu, uw
	- Description :The ID of a data contributor. Identifies the network considered to be the preferred source of information for this event.


** nph **
	- Number of Phases Used : String
	- Description : Number of P and S arrival-time observations used to compute the hypocenter location. Increased numbers of arrival-time observations generally result in improved earthquake locations.



** nst **
	- Data Type : Integer
	- Description : The total number of seismic stations used to determine earthquake location.
	- Additional Information :Number of seismic stations which reported P- and S-arrival times for this earthquake. This number  : ay be larger than the Number of Phases Used if arrival times are rejected because the distance to a seismic station exceeds the maximum allowable distance or because the arrival-time observation is inconsistent with the solution.



** place **
	- Data Type : String
	- - Description : Textual description of named geographic region near to the event. This may be a city name, or a Flinn-Engdahl Region name.
	- Additional Information : We use a GeoNames dataset to reference populated places that are in close proximity to a seismic event. GeoNames has compiled a list of cities in the United States where the population is 1,000 or greater (cities1000.txt). This is the primary list that we use when selecting nearby places. In order to provide the public with a better understanding for the location of an event we try to list a variety of places in our nearby places list. This includes the closest known populated place in relation to the seismic event (which based on our dataset will have a population of 1,000 or greater). We also include the next 3 closest places that have a population of 10,000 or greater, and finally make sure to include the closest capital city to the seismic event. The reference point for the descriptive locations is usually either the City Hall of the town (or prominent intersection in the middle of town if there is no City Hall), but please refer to the GeoNames website for the most accurate information on their data. If there is no nearby city within 300 kilometers (or if the nearby cities database is unavailable for some reason), the Flinn-Engdahl (F-E) seismic and geographical regionalization scheme is used. The boundaries of these regions are defined at one-degree intervals and therefore differ from irregular political boundaries. For example, F-E region 545 (Northern Italy) also includes small parts of France, Switzerland, Austria and Slovenia and F-E region 493 (Chesapeake Bay Region) includes all of the State of Delaware, plus parts of the District of Columbia, Maryland, New Jersey, Pennsylvania and Virginia. Beginning with January 2000, the 1995 revision to the F-E code has been used in the QED and PDE listings. As an agency of the U.S. Government, we are expected to use the names and spellings approved by the U.S. Board on Geographic Names. Any requests to approve additional names should be made to the U.S. Board on Geographic Names.



** rms **
	- Data Type : Decimal
	- Typical Values : [0.13,1.39]
	- Description :The root-mean-square (RMS) travel time residual, in sec, using all weights. This parameter provides a measure of the fit of the observed arrival times to the predicted arrival times for this location. Smaller numbers reflect a better fit of the data. The value is dependent on the accuracy of the velocity model used to compute the earthquake location, the quality weights assigned to the arrival time data, and the procedure used to locate the earthquake.



** sig **
	- Data Type : Integer
	- Typical Values : [0, 1000]
	- Description :A number describing how significant the event is. Larger numbers indicate a more significant event. This value is determined on a number of factors, including: magnitude, maximum MMI, felt reports, and estimated impact.



** sources **
	- Data Type : String
	- Typical Values : ",us,nc,ci,"
	- Description :A comma-separated list of network contributors.



** status **
	- Data Type : String
	- Typical Values : “automatic”, “reviewed”, “deleted”
	- Description :Indicates whether the event has been reviewed by a human.
	- Additional Information : Status is either automatic or reviewed. Automatic events are directly posted by automatic processing systems and have not been verified or altered by a human. Reviewed events have been looked at by a human. The level of review can range from a quick validity check to a careful reanalysis of the event.



** time **
	- Data Type : Long Integer
	- Description : Time when the event occurred. Times are reported in milliseconds since the epoch ( 1970-01-01T00:00:00.000Z), and do not include leap seconds. In certain output formats, the date is formatted for readability.
	- Additional Information : We indicate the date and time when the earthquake initiates rupture, which is known as the "origin" time. Note that large earthquakes can continue rupturing for many 10's of seconds. We provide time in UTC (Coordinated Universal Time). Seismologists use UTC to avoid confusion caused by local time zones and daylight savings time. On the individual event pages, times are also provided for the time at the epicenter, and your local time based on the time your computer is set.



** tsunami **
	- Data Type : Integer
	- Description : This flag is set to "1" for large events in oceanic regions and "0" otherwise. The existence or value of this flag does not indicate if a tsunami actually did or will exist. If the flag value is "1", the event will include a link to the NOAA Tsunami website for tsunami information. The USGS is not responsible for Tsunami warning; we are simply providing a link to the authoritative NOAA source. :See http://www.tsunami.gov/ for all current tsunami alert statuses.



** type **
	- Data Type : String
	- Typical Values : “earthquake”, “quarry”
	- Description :Type of seismic event.



** types **
	- Data Type : String
	- Typical Values : “,cap,dyfi,general-link,origin,p-wave-travel-times,phase-data,”
	- Description :A comma-separated list of product types associated to this event.



** tz **
	- Data Type : Integer
	- Typical Values : [-1200, +1200]
	- Description :Timezone offset from UTC in minutes at the event epicenter.



** updated **
	- Data Type : Long Integer
	- Description : Time when the event was most recently updated. Times are reported in milliseconds since the epoch. In certain output formats, the date is formatted for readability. :


** url **
	- Data Type : String
	- - Description : Link to USGS Event Page for event.

Set the path to the working directory


```python
pathToWorkingFolder = 'E:\Development\Quake\iPNY'
```

##### Uncomment and execute the below cell to install required libraries if not already installed.


```python
#!pip install numpy requests pandas folium requests_futures
```

Importing required libraries and dependencies


```python
import datetime, json, requests, pandas as pd, numpy as np, folium, os
from IPython.display import HTML, display
from folium.plugins import MarkerCluster
from requests.packages.urllib3.exceptions import InsecureRequestWarning
from dateutil import relativedelta
from requests_futures.sessions import FuturesSession
from concurrent.futures import wait
from datetime import datetime,timedelta
from pandas.io.json import json_normalize
from pandas.plotting import scatter_matrix
#import EQClass

#to supress the warning due to unvarified SSl during HTTP request.
requests.packages.urllib3.disable_warnings(InsecureRequestWarning)

#Path to the working folder
os.chdir(pathToWorkingFolder)
print('Current Working Dir:', os.getcwd())
```

    Current Working Dir: E:\Development\Quake\iPNY
    

Create a class to invoke requests to get data from USGS, process them and return the refined data


```python
import pandas as pd
import json, requests
from pandas.io.json import json_normalize
from requests.packages.urllib3.exceptions import InsecureRequestWarning

class EQData:
   'Common base class to invoke HTTP requests and get data from the earthquake.usgs.gov website'
   
   def getDateList(self, start,end):
        startTime = start
        endTime = end
        startDateList = [startTime]
        endDateList = []
        dateFormat = '%Y-%m-%d'
        startDate = datetime.strptime(startTime, dateFormat)
        endDate = datetime.strptime(endTime, dateFormat)
        datelist = pd.date_range(startDate, endDate, freq='M')

        for element in datelist:
            element = element + timedelta(days=1)
            startDateList.append(element.strftime(dateFormat) )
            endDateList.append(element.strftime(dateFormat))
        return startDateList,endDateList

   def downloadRawData(self, startDate, endDate):
     try:
         isRawDataAvailable = False
         startDate = startDate
         endDate = endDate
         requests.packages.urllib3.disable_warnings(InsecureRequestWarning)
         data = requests.get('https://earthquake.usgs.gov/fdsnws/event/1/query?format=geojson&starttime='+startDate+'&endtime='+endDate, verify=False)
         self.rawData = data.content
         isRawDataAvailable = True
         return isRawDataAvailable
     except:
         return isRawDataAvailable 
    
   def parseRawDataToJson(self):
     try:
         isDataParsedToJson = False
         self.geoJsonData = json.loads(self.rawData)
         isDataParsedToJson = True
         return isDataParsedToJson 
     except:
         return isDataParsedToJson 
     
   def getProcessedData(self):
        try:
            isProcessedDataAvailable = False
            df = json_normalize(self.geoJsonData['features'])
            df[['Longitude','Latitude','Depth']] = pd.DataFrame(df['geometry.coordinates'].values.tolist(), index= df.index)
            df = df.rename(columns=lambda x: x.replace('geometry.', '').replace('properties.', ''))
            df.drop(columns=['coordinates','type', 'code','detail','ids', 'types'], inplace=True)
            df.time = pd.to_datetime(df.time , unit='ms')
            df.updated = pd.to_datetime(df.updated , unit='ms')
            df = df.rename(str.upper, axis='columns')            
            self.RefinedData = df
            isProcessedDataAvailable = True
            return isProcessedDataAvailable
        except:
            return isProcessedDataAvailable
    
   def getEQBulkParallelData(self,startDate,endDate):
        #try:
            isBulkProcessedDataAvailable = False
            startTime = datetime.now()
            start = datetime.now()
            startDateList,endDateList = self.getDateList(startDate,endDate)
            requestUrls = ['https://earthquake.usgs.gov/fdsnws/event/1/query?format=geojson&eventtype=earthquake&starttime='+st+'&endtime='+end for st,end in zip(startDateList,endDateList)]

            session = FuturesSession(max_workers=len(requestUrls))

            dataMerged = pd.DataFrame()

            for i in range(len(requestUrls)):
                 #response[i] = session.get(i).result().content
                 response = session.get(requestUrls[i],verify=False).result().content

                 df = json_normalize(json.loads(response)['features'])
                 df[['Longitude','Latitude','Depth']] = pd.DataFrame(df['geometry.coordinates'].values.tolist(), index= df.index)
                 df = df.rename(columns=lambda x: x.replace('geometry.', '').replace('properties.', ''))
                 df.drop(columns=['coordinates','type', 'code','detail','ids', 'types'], inplace=True)
                 df.time = pd.to_datetime(df.time , unit='ms')
                 df.updated = pd.to_datetime(df.updated , unit='ms')
                 df = df.rename(str.upper, axis='columns')            
                 dataMerged = dataMerged.append(df)
            print(calculateTimeTaken('parallel EQ data downloading',startTime,datetime.now()))
        #   file = open('data\\'+startDateList[i]+".txt",'w')
        #   file.write(str(response))
        #   file.close()
        #   dataMerged.to_csv('MergedData.csv',index=False)
            self.BulkProcessedData = dataMerged
            isBulkProcessedDataAvailable = True
            return isBulkProcessedDataAvailable
        #except:
           # return isBulkProcessedDataAvailable
```

##### Generic function to calculate time taken to complete a code snippet


```python
def calculateTimeTaken(processName, startTime, endTime):
    elaspsedTime = str(endTime-startTime)
    elaspsedTime = elaspsedTime.split(':')
    return 'Total time taken to complete %s : %s Mins %s Seconds'% (processName, elaspsedTime[1],elaspsedTime[2][0:2])

```

##### Code to color the markers on the map basis the magnitude of the Earthquake


```python
def iconColor(mag):
    if( mag < 2 ):
        return 'green'
    elif( 2 <= mag < 3 ):
        return 'orange'
    elif( 3 <= mag < 4.5 ):
        return 'red'
    else:
        return 'darkred'
```

##### Function to generate geo map of given dataframe and save it to a HTML file


```python
def generateMap(dataFrame, mapPath):
    try:
        startTime = datetime.now()
        map=folium.Map(location=[dataFrame['LATITUDE'].mean(),dataFrame['LONGITUDE'].mean()],zoom_start=4,tiles="Mapbox Bright")
        fg=folium.FeatureGroup(name="EarthQuakes")
        mc = MarkerCluster()
        for lt, ln, dp, mag, tp, st, ti, tl, url in zip(dataFrame['LATITUDE'], dataFrame['LONGITUDE'], dataFrame['DEPTH'], dataFrame['MAG'],dataFrame['MAGTYPE'],dataFrame['STATUS'],dataFrame['TIME'],dataFrame['TITLE'],dataFrame['URL']):
            popupHTML = """<div style="background: """+iconColor(mag)+""";"><a target="_blank" href="""+url+""">
                           <h4>"""+str(tl).replace("'","")+"""</h4>
                           </a>Magnitude : """+str(mag)+" "+str(tp)+"""
                           <br>Depth : """+str(dp)+"""
                           <br>Latitude : """+str(lt)+"""
                           <br>Longitude : """+str(ln)+"""
                           <br>Date Time : """+str(ti)+"""</div>"""
            mc.add_child(folium.Marker(location=[lt, ln],popup=popupHTML, icon=folium.Icon(color=iconColor(mag))))
        map.add_child(mc)
        map.save(mapPath)
        print(calculateTimeTaken('geo co-ordinates mapping',startTime,datetime.now()))
        return map
    except:
        print('Error occured while generating Map')
        pass
```

##### Download sample eaarthquake data since 1st Jan'16 to 2nd Jan'16
_For exploratory purpose and to make sure our classes and functions are working fine, we will download sample data fron USGS API and post processing and converting, will store it in a pandas dataframe._


```python
startDate = '2016-01-01'
endDate = '2016-01-02'

if(__name__=='__main__'):
    try:        
        
        startTime = datetime.now()
        objEQData = EQData()
        #objEQData = EQClass.EQData()
        isRawDataAvailable = objEQData.downloadRawData(startDate , endDate )

        if(isRawDataAvailable):
            print('Raw Data Downloaded Successfully')
            isParseDataAvailable = objEQData.parseRawDataToJson()
            print(calculateTimeTaken('data download',startTime,datetime.now()))
        
        if(isParseDataAvailable):
            startTime = datetime.now()
            print('Raw Data Parsed To JSON Successfully')
            isProcessedDataAvailable = objEQData.getProcessedData()
            print(calculateTimeTaken('data parsing',startTime,datetime.now()))
        
        if(isProcessedDataAvailable):
            startTime = datetime.now()
            print('JSON Data Processed Successfully To DataFrame')
            processedDataFrame = objEQData.RefinedData
            processedDataFrame.to_csv('SampleData.csv')
            print(calculateTimeTaken('data processing',startTime,datetime.now()))
    except Exception as e:
        print('Error occured in the main process. Please confirm the details and re-process')
        pass

```

    Raw Data Downloaded Successfully
    Total time taken to complete data download : 00 Mins 02 Seconds
    Raw Data Parsed To JSON Successfully
    Total time taken to complete data parsing : 00 Mins 00 Seconds
    JSON Data Processed Successfully To DataFrame
    Total time taken to complete data processing : 00 Mins 00 Seconds
    

##### Generate map of sample eaarthquake data since 1st Jan'16 to 2nd Jan'16
_For exploratory purpose we will plot all the geo-cordinates on a map._


```python
mapSample = generateMap(processedDataFrame, "mapSample.html")
display(mapSample)
```

    Total time taken to complete geo co-ordinates mapping : 00 Mins 03 Seconds
    

##### Download all the eaarthquake data since 1st Jan'16 to 31st Dec'16
_Since the dataset is pretty huge and we can not download all the data in a single call, we will use the parallel HTTP calls to the USGS APIs and eventually merge all the data in a single datafrmae._


```python
startDate = '2016-01-01'
endDate = '2016-12-31'
if(__name__=='__main__'):
    try:
        
        objEQData = EQData()
        isBulkProcessedDataAvailable = objEQData.getEQBulkParallelData(startDate, endDate)
        
        if(isBulkProcessedDataAvailable):
            processedBulkDataFrame = objEQData.BulkProcessedData
            processedBulkDataFrame.to_csv('BulkData.csv')            
    except Exception as e:
        print('Error occured in the main process. Please confirm the details and re-process')
        pass
```

    Total time taken to complete parallel EQ data downloading : 01 Mins 34 Seconds
    

##### Get first 5 records from the downloaded bulk data set


```python
processedBulkDataFrame.head()
#processedBulkDataFrame.columns
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>ID</th>
      <th>ALERT</th>
      <th>CDI</th>
      <th>DMIN</th>
      <th>FELT</th>
      <th>GAP</th>
      <th>MAG</th>
      <th>MAGTYPE</th>
      <th>MMI</th>
      <th>NET</th>
      <th>...</th>
      <th>STATUS</th>
      <th>TIME</th>
      <th>TITLE</th>
      <th>TSUNAMI</th>
      <th>TZ</th>
      <th>UPDATED</th>
      <th>URL</th>
      <th>LONGITUDE</th>
      <th>LATITUDE</th>
      <th>DEPTH</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>nc72586936</td>
      <td>None</td>
      <td>NaN</td>
      <td>0.002703</td>
      <td>NaN</td>
      <td>78.000000</td>
      <td>0.19</td>
      <td>md</td>
      <td>NaN</td>
      <td>nc</td>
      <td>...</td>
      <td>automatic</td>
      <td>2016-01-31 23:55:53.870</td>
      <td>M 0.2 - 10km WNW of The Geysers, California</td>
      <td>0</td>
      <td>-480.0</td>
      <td>2017-02-09 21:36:11.801</td>
      <td>https://earthquake.usgs.gov/earthquakes/eventp...</td>
      <td>-122.854332</td>
      <td>38.825165</td>
      <td>2.40</td>
    </tr>
    <tr>
      <th>1</th>
      <td>ak12603559</td>
      <td>None</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>100.799992</td>
      <td>0.10</td>
      <td>ml</td>
      <td>NaN</td>
      <td>ak</td>
      <td>...</td>
      <td>reviewed</td>
      <td>2016-01-31 23:48:17.000</td>
      <td>M 0.1 - 20km SSW of Ester, Alaska</td>
      <td>0</td>
      <td>-540.0</td>
      <td>2016-02-06 05:17:15.628</td>
      <td>https://earthquake.usgs.gov/earthquakes/eventp...</td>
      <td>-148.123100</td>
      <td>64.666600</td>
      <td>10.30</td>
    </tr>
    <tr>
      <th>2</th>
      <td>us20004w7m</td>
      <td>None</td>
      <td>1.0</td>
      <td>0.527000</td>
      <td>0.0</td>
      <td>50.000000</td>
      <td>4.00</td>
      <td>mb</td>
      <td>NaN</td>
      <td>us</td>
      <td>...</td>
      <td>reviewed</td>
      <td>2016-01-31 23:47:11.160</td>
      <td>M 4.0 - 23km E of Lalas, Greece</td>
      <td>0</td>
      <td>120.0</td>
      <td>2016-04-14 22:51:38.040</td>
      <td>https://earthquake.usgs.gov/earthquakes/eventp...</td>
      <td>21.978500</td>
      <td>37.705200</td>
      <td>10.00</td>
    </tr>
    <tr>
      <th>3</th>
      <td>ak12714383</td>
      <td>None</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>237.599981</td>
      <td>2.40</td>
      <td>ml</td>
      <td>NaN</td>
      <td>ak</td>
      <td>...</td>
      <td>reviewed</td>
      <td>2016-01-31 23:43:22.000</td>
      <td>M 2.4 - 95km SE of Cold Bay, Alaska</td>
      <td>0</td>
      <td>-660.0</td>
      <td>2016-02-06 05:17:14.613</td>
      <td>https://earthquake.usgs.gov/earthquakes/eventp...</td>
      <td>-161.748600</td>
      <td>54.533600</td>
      <td>21.60</td>
    </tr>
    <tr>
      <th>4</th>
      <td>mb80122939</td>
      <td>None</td>
      <td>NaN</td>
      <td>0.225000</td>
      <td>NaN</td>
      <td>119.000000</td>
      <td>1.04</td>
      <td>ml</td>
      <td>NaN</td>
      <td>mb</td>
      <td>...</td>
      <td>reviewed</td>
      <td>2016-01-31 23:42:44.890</td>
      <td>M 1.0 - 28km SSE of Virginia City, Montana</td>
      <td>0</td>
      <td>-420.0</td>
      <td>2016-02-02 00:08:46.840</td>
      <td>https://earthquake.usgs.gov/earthquakes/eventp...</td>
      <td>-111.869833</td>
      <td>45.044000</td>
      <td>3.04</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 25 columns</p>
</div>



##### Lets see some of the general statistics of each variable in the dataset.
_This will give us the count, mean, std, min and max value of each variable_


```python
processedBulkDataFrame.describe().round(2)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>CDI</th>
      <th>DMIN</th>
      <th>FELT</th>
      <th>GAP</th>
      <th>MAG</th>
      <th>MMI</th>
      <th>NST</th>
      <th>RMS</th>
      <th>SIG</th>
      <th>TSUNAMI</th>
      <th>TZ</th>
      <th>LONGITUDE</th>
      <th>LATITUDE</th>
      <th>DEPTH</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>16029.00</td>
      <td>85669.00</td>
      <td>16029.00</td>
      <td>95377.00</td>
      <td>121660.00</td>
      <td>1086.00</td>
      <td>76339.00</td>
      <td>122050.00</td>
      <td>122108.00</td>
      <td>122108.00</td>
      <td>119395.00</td>
      <td>122108.00</td>
      <td>122108.00</td>
      <td>122108.00</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>1.61</td>
      <td>0.74</td>
      <td>14.20</td>
      <td>125.40</td>
      <td>1.68</td>
      <td>3.84</td>
      <td>17.40</td>
      <td>0.33</td>
      <td>70.30</td>
      <td>0.00</td>
      <td>-374.19</td>
      <td>-105.67</td>
      <td>38.69</td>
      <td>28.46</td>
    </tr>
    <tr>
      <th>std</th>
      <td>1.10</td>
      <td>2.42</td>
      <td>522.58</td>
      <td>68.31</td>
      <td>1.32</td>
      <td>1.59</td>
      <td>15.05</td>
      <td>0.31</td>
      <td>105.37</td>
      <td>0.06</td>
      <td>290.03</td>
      <td>76.43</td>
      <td>21.96</td>
      <td>61.35</td>
    </tr>
    <tr>
      <th>min</th>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>7.00</td>
      <td>-1.90</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>-720.00</td>
      <td>-180.00</td>
      <td>-79.98</td>
      <td>-6.60</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>1.00</td>
      <td>0.03</td>
      <td>0.00</td>
      <td>75.00</td>
      <td>0.77</td>
      <td>3.25</td>
      <td>7.00</td>
      <td>0.09</td>
      <td>9.00</td>
      <td>0.00</td>
      <td>-480.00</td>
      <td>-148.92</td>
      <td>34.04</td>
      <td>4.10</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>1.00</td>
      <td>0.08</td>
      <td>0.00</td>
      <td>109.97</td>
      <td>1.30</td>
      <td>3.90</td>
      <td>13.00</td>
      <td>0.20</td>
      <td>26.00</td>
      <td>0.00</td>
      <td>-420.00</td>
      <td>-120.61</td>
      <td>38.79</td>
      <td>9.61</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>2.00</td>
      <td>0.28</td>
      <td>1.00</td>
      <td>159.10</td>
      <td>2.14</td>
      <td>4.46</td>
      <td>22.00</td>
      <td>0.52</td>
      <td>69.00</td>
      <td>0.00</td>
      <td>-420.00</td>
      <td>-116.29</td>
      <td>57.38</td>
      <td>21.60</td>
    </tr>
    <tr>
      <th>max</th>
      <td>9.10</td>
      <td>56.90</td>
      <td>60431.00</td>
      <td>360.00</td>
      <td>7.90</td>
      <td>9.40</td>
      <td>236.00</td>
      <td>8.92</td>
      <td>2910.00</td>
      <td>1.00</td>
      <td>780.00</td>
      <td>180.00</td>
      <td>86.33</td>
      <td>673.06</td>
    </tr>
  </tbody>
</table>
</div>



##### Generate map of all the eaarthquake data since 1st Jan'16 to 31st Dec'16. This gonna take really long time.
_Since the dataset is pretty huge and there are lot of geo-cordinates to plot, this code snippet is gonna take really long time to run. In my experimentations, it took around 20 mins to generate and save the map._


```python
startTime = datetime.now()
mapBulk = generateMap(processedBulkDataFrame, "mapBulk.html")
print(calculateTimeTaken('bulk data mapping',startTime,datetime.now()))
```

    Total time taken to complete geo co-ordinates mapping : 34 Mins 08 Seconds
    Total time taken to complete bulk data mapping : 34 Mins 08 Seconds
    

##### Display the map of all the eaarthquake data since 1st Jan'16 to 31st Dec'16
_Since the generated map is pretty big as there are lot of geo-cordinates, the iframe is not gonna render in the notebooks. We will open the map in a seperate tab in the chrome browser_


```python
#display(mapBulk)
```

##### Display the histograms of all the variables


```python
processedBulkDataFrame.hist(column = ['LONGITUDE', 'LATITUDE','DEPTH','CDI', 'DMIN', 'FELT', 'GAP', 'MAG', 'MMI','NET', 'NST', 'RMS', 'SIG',  'TSUNAMI'], figsize=(40,40),bins= 25)
```




    array([[<matplotlib.axes._subplots.AxesSubplot object at 0x0000020748482A58>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x00000207486A99B0>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x00000207486E4A20>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x000002074871DA20>],
           [<matplotlib.axes._subplots.AxesSubplot object at 0x000002074874DA20>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x000002074874DA58>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x00000207487C0B38>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x00000207487F8BA8>],
           [<matplotlib.axes._subplots.AxesSubplot object at 0x000002074882E128>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x0000020748868128>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x0000020750C5D668>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x0000020750C956D8>],
           [<matplotlib.axes._subplots.AxesSubplot object at 0x0000020750CD25F8>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x0000020750CFAB38>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x0000020750D34B38>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x0000020750D6AA58>]],
          dtype=object)




![png](output_34_1.png)


##### Display the co-relation matrix of all the variables


```python
corr = processedBulkDataFrame.corr()
corr.style.background_gradient().set_precision(2)
```



_The above plot shows how each variable is co-releated with other variables. We can see some variables are highly corelated like_
- SIG and MAG are +vely co-related with a coeffecient of 94%
- SIG and LONGITUDE are +vely co-related with a coeffecient of 68%
- SIG and LATITUDE are -vely/inversely co-related with a coeffecient of 63%
- MAG and LONGITUDE are +vely co-related with a coeffecient of 61% 
- TZ and LONGITUDE are obviously +vely co-related with a coeffecient of 97%


```python
subsetBulk = processedBulkDataFrame[['TIME','LATITUDE','LONGITUDE','MAG','DEPTH']]
```


```python
groupedByTime = subsetBulk.groupby(subsetBulk.TIME.dt.hour).mean()
```


```python
groupedByTime.plot.line(y='LATITUDE')
groupedByTime.plot.line(y='LONGITUDE')
groupedByTime.plot.line(y='DEPTH')
groupedByTime.plot.line(y='MAG')
```




    <matplotlib.axes._subplots.AxesSubplot at 0x2075032e518>




![png](output_40_1.png)



![png](output_40_2.png)



![png](output_40_3.png)



![png](output_40_4.png)



```python
groupedByMonth = subsetBulk.groupby(subsetBulk.TIME.dt.month).mean()
```


```python
groupedByMonth.plot.line(y='LATITUDE')
groupedByMonth.plot.line(y='LONGITUDE')
groupedByMonth.plot.line(y='DEPTH')
groupedByMonth.plot.line(y='MAG')
```




    <matplotlib.axes._subplots.AxesSubplot at 0x207504149b0>




![png](output_42_1.png)



![png](output_42_2.png)



![png](output_42_3.png)



![png](output_42_4.png)

