# Haze

There are 2 main problems that most existing projects try to address:
1. Real-time measures: What is the current air quality?
2. Forecasting: How might this change in the coming day(s)? Should an upcoming outdoor event be cancelled? 


## Data Sources

### Official (NEA)

#### Real-time measurements

- [1hr PM2.5](https://data.gov.sg/datasets/d_e1058d6974c877257e32048ab128ad83/view) (concentration, in µg/m3) - website claims "Updated every 15 minutes" but from observation it seems to be updated only every hour despite `updatedTimestamp` changing every 15 minutes. So unfortunately, it isn't *that* real-time. Note that this is **NOT an instantaneous reading**, it's an average over the past 60 minutes but its underlying sampling frequency is unclear. 
- [24hr PSI](https://data.gov.sg/datasets/d_fe37906a0182569d891506e815e819b7/view) - same issue as above; provides overall PSI readings as well as its constituents, out of which PM10 is probably of more interest since the others are often way below WHO guidelines.

Additional notes (for both endpoints)
1. Granularity is limited to 5 regions (North, South, East, West, Central), but only an arbitrary lat/long coordinate is provided for each region (i.e. no details about how boundary is drawn). 
2. This gives "real-time" data as well as historical data, but for the latter it's probably more convenient to grab them [here](https://data.gov.sg/datasets?q=&query=pollutant&groups=&organization=&resultId=d_ec484d9610a22c57fa873b14f9baa57a).
3. Data provenance: Unclear
    - This [paper](https://www.mdpi.com/1660-4601/19/13/7728) mentions "22 air quality monitoring stations (18 monitor general ambient air quality and 4 monitor roadside air quality)" and cited this [2018 Annual Report from NEA](https://www.nea.gov.sg/docs/default-source/resource/publications/environmental-protection-division-annual-report/epd-report-2018-v4-(compressed).pdf). This [figure](https://www.researchgate.net/figure/Locations-of-22-air-quality-AQ-monitoring-stations-18-monitor-general-air-quality-and_fig2_361547149) gives you the location of the stations. However, they did not mention whether the 5 readings were derived from them (but they likely used the same 5-region data as what the API provides). So it remains hazy until someone from NEA comes to clear the air! 
    - Here's an [image](https://www.facebook.com/photo/?fbid=1281698854104527&set=a.164266095847814) of how one of the air quality monitoring stations looks like.
4. Uptime: Generally reliable. There are rather frequently occurring maintenance windows (e.g. weekly-fortnightly) but the API remains available in some of these periods. 


#### Forecast

As of 17/10/26, NEA's haze microsite now shows this [in an image](https://www.haze.gov.sg/images/default-source/health-advisory-images/new-chart.png) with better quantification (e.g. next day PSI of 150-230) than before. 


### Community-sourced

#### Real-time measurements

**AirGradient**

(tbc!)

**AQICN** 

A single endpoint provides these air quality measurements (along with other weather information such as temperature, relative humidity, pressure, wind speed):
1. PM2.5 (scaled to AQI) - instantaneous reading updated every ~3 minutes with a 5-minute lag from current time
2. PM10 - same as above
3. PM1 - same as above


Additional notes
1. Getting the data *officially* involves accessing the API [here](https://aqicn.org/api/) and obtaining a token (might take 15 minutes for it to be activated after you get their email). API limits are very generous and more than sufficient for SG. 
    - Station IDs can be obtained by tapping on the station in the [map](https://aqicn.org/). 
    - Strangely, you need to [use A instead of @](https://github.com/home-assistant/core/issues/96865#issuecomment-1685421228) when setting the station ID parameter of the endpoint but this is actually documented on their website.
2. If you plot the data (or just observe from the plot shown in the website) you will note that **some stations have weird oscillations**, so taking their reading at face value is not a good idea. In such cases, it will be better to use some form of moving average. 
3. Data Provenance
    1. Generally, they are from sensors that are likely cheaper and less sophisticated than what NEA uses. However, they give you more granularity in both space and time - you get a reading from a precise location and it gets updated every ~3 minutes. 
    2. The device looks like [this](https://aqicn.org/gaia/).
    3. Since it's handled by non-experts, it is not always reliable. Even though some [QC is done](https://aqicn.org/station/validation/@2791069/), it doesn't stop showing the station when it's reporting 500 AQI and the neighbours aren't. **You need to do your own sanity checks and filtering**. 
    4. [Their methodologies vary](https://www.channelnewsasia.com/singapore/haze-psi-aqi-air-quality-pm25-6452896) so you have to be careful:
        1. The sensors actually record raw concentration (i.e. same units as what NEA reports), but the website [converts](https://aqicn.org/calculator/) it to AQI via [`instantcast`](https://aqicn.org/faq/2015-03-15/air-quality-nowcast-a-beginners-guide/). AQI follows a difference scale. You can easily undo this to get the raw numbers back, it's just 'segmented' linear interpolation (i.e. AQI scale is divided into bins because we want to determine which range of values is considered good/unhealthy/hazardous. Each bin has a range of concentration (e.g. 35.5-55.4) and a range of AQI (e.g. 101-150), so if the concentration is in the middle of the range i.e., 45, you simply get AQI of ~125 because that's the middle too).
        2. Basically, the official concentration-to-AQI conversion uses a 24-hour window which clearly is not what laypeople like (people get angry when what they experience with their own eyes/nose is not reflected in the data; by the time the AQI alerts us we already took in hours of smoke - too late!). But for health advisories, the scientific studies they are based on use such windows. There are now studies that use smaller windows but it will take some time for *science* to come to a consensus so it's down to individuals to make informed decisions (i.e. official bodies aren't likely to make recommendations *not backed by science* and take responsibility for it) for ourselves.
4. Uptime: Usually good, but their website can go down when the whole SG population is visiting their website (e.g. it was unconscious for a while on 17 Oct 2026 afternoon). Their API remains strong though!


### Others

#### Real-time measurements

1. \[Updated hourly\] Aviation uses weather information too - while it doesn't give air quality measures, the [visibility plot](https://metar-taf.com/metar/WSSS) is a useful alternative if other information sources are down. It's only for Changi, PLAB and Seletar though. 


#### Forecasting

1. [MyCMAQ](https://mycmaq.met.gov.my/) - Singapore looks really small here so it's not possible to get granular forecasts here but it gives a great macro **5-day lookahead forecast via heatmaps**. It is quite complicated (seeded by Global Forecast System, considers atmospheric dynamics, chemical processes, industrial emissions, burning emissions) so probably better than what indie efforts can produce even with agentic powers.
2. [ASMC](https://asmc.asean.org/home/) - 3-day lookahead but it allows zooming and considers rainfall. Also has a 1-day lookahead particularly for [surface PM10](https://wis2.asmc.asean.org/smoke-haze-dispersion-model/) updated every 3 hours.




## Project Examples

> Projects with unique features not in official sources


[hazynot.sg](https://hazynot.sg/)
- Multi-day forecast (based on satellite imaging of fires and smoke, wind and rain)
- Did backtesting! 

[hazesg.fyi](https://hazesg.fyi/)
- Chinese translations, larger font size for seniors
- 12 hour forecast
- CADR calculator 

[hazenow.app](https://hazenow.app/)
- 5-day forecast (derived from [Open-Meteo](https://open-meteo.com/) and [WeatherNext 3](https://deepmind.google/science/weathernext/))
- Trajectory showing where the haze is coming from 


[Haze@SG app](https://apps.apple.com/sg/app/haze-sg/id663816616)
- Has haze-related news

### Other fun projects

- [Online sentiment analysis](https://jadangpooiling.medium.com/listening-to-haze-using-sentimental-analytics-to-understand-public-response-in-malaysia-singapore-d1cff86a676e) - sentiment analysis on YouTube comments and Google Trends; see also a [2015 study from SMU](https://ink.library.smu.edu.sg/cgi/viewcontent.cgi?params=/context/sis_research/article/5397/&path_info=a_study_on_sg_haze.pdf)

- [Images of haze](https://walaohaze.com/) - crowdsourcing how hazy skies look like in different parts of SG

- [Planet Out Loud](https://www.planetoutloud.com/) - a Three.js visualisation of the NUS campus with PM2.5 readings from AirGradient sensors

### Interesting information

- [Since 2024](https://www.straitstimes.com/singapore/ntu-scientists-peer-into-upper-atmosphere-to-get-a-timelier-more-accurate-forecast-of-haze), Earth Observatory of Singapore (in NTU) actually has Doppler Lidar monitoring stations which can forecast haze earlier and overcome cloud cover limitations in satellite imaging approaches. It does so by monitoring the upper atmosphere instead of ground level (which is what most air quality monitors look at). They are located at NTU, Woodlands and RGS and there's a [dashboard](https://lidar-dashboard-3dreams.vercel.app/dashboard) that shows you whether there are pollutants up to 3km above and also the vertical wind speed. 
- First notable haze incident in SG was in [October 1961](https://www.straitstimes.com/singapore/environment/55-years-of-haze-how-plane-bound-for-kl-overflew-to-singapore-when-it-was-all)


## Gaps

1. Accuracy of the forecasts is not often well quantified
2. Make community-sourced readings more robust (but AQICN doesn't allow their data to be "redistributed as cached or archived data" 🤷)


## Acknowledgements

Many of these links above are collated from [Reddit](https://www.reddit.com/r/singapore/) discussions, supplemented by my own research and experiences from using the APIs. Hopefully putting these information together makes it clearer what has been done! It is already quite saturated. 
