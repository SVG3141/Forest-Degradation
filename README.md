# Forest-Degradation

Forestry has been one of Ontario’s main natural resource industries since the end of the Second World War.  The boreal forest makes up 50% of all land within the province, and 58% of the boreal forest is zoned as an “area of the undertaking” which allows economic activity such as forestry and mining (Ter-Mikaelian et al., 2021). Any forestry activity that takes place within the area of the undertaking is regulated by the Crown Forest Sustainability Act, ON (1994). This policy mandates the reforestation of harvested areas and is intended to ensure the long-term health of Ontario’s forests and ecosystems through both anthropogenic and natural regeneration. However, scientific research has highlighted the impacts of economic activity on woodland caribou survivability, habitat fragmentation, forest degradation and carbon sequestration in the boreal ecosystem under the regulation of this policy (Betts et al., 2022; Fryxell et al., 2020; Harris et al., 2021; Vazquez-Grandon et al., 2018). This research asks the question; can forest degradation in Ontario’s boreal forest be identified through remote sensing? Furthermore, this research will also attempt to answer the sub-question; can forest degradation patterns in managed areas be differentiated from natural disturbance regimes in unmanaged forests? Forest degradation can be defined as any reduction in a forest’s ecological integrity, including ecosystem resilience, caused by direct or indirect anthropogenic activity and impact (Vasquez-Grandon et al., 2018).<br />
Keywords: Forest Degradation, Random Forest Learning Model, Ontario, Remote Sensing <br />
<br />

## Methods <br />
### Preprocessing <br />

The downloaded Landsat images from the USGS GloVis platform are rasters containing DN (digital number) values. DN values are numbers representing the brightness value of a pixel in a digital image. For example, in Landsat 5, the remote sensing images are 8-bit images and the DN value ranges from 1-255.<br />

To calculate the spectrometric band-based indices such as NDVI, the DN value needs to be first converted into radiance, and then into reflectance. According to Chander et al. (2007), the process of obtaining radiance from Landsat 5 product’s DN value is given by the following relationship:<br />


where<br />

Lλ  is the calculated spectral radiance [W/(m2·sr·㎛)]<br />

Qcal = calibrated pixel value, also known as digital number [DN]<br />

LMINλ = spectral radiance scales to Qcal min [W/(m2·sr·㎛)]<br />

LMAXλ = spectral radiance scales to Qcal max [W/(m2·sr·㎛)]<br />

Qcal min = minimum quantized calibrated pixel value (typically = 1)<br />

Qcal max = maximum quantized calibrated pixel value (typically = 255)<br />

In this calculation, LMINλ, LMAXλ, Qcal min, Qcal max values were extrapolated from the metadata of the downloaded image. Once again, the spatial analyst tool Raster calculator in ArcGIS Pro was then applied for the conversion from DN value to radiance.<br />

After calculating the radiance, one can further calculate the ToA (top of atmosphere) reflectance from the radiance value Lλ. Reflectance is the ratio of radiance (radiation leaving the Earth) over irradiance (radiation reaching the Earth from the sun), and can be calculated by the following equation from the Landsat 7 Users Handbook – Chapter 11:<br />


where<br />

ρλ = Unitless planetary reflectance <br />

Lλ= calculated spectral radiance (from earlier step) [W/(m2·sr·㎛)]<br />

d = Earth-Sun distance in astronomical units<br />

ESUNλ = mean solar exoatmospheric irradiances [W/(m2·㎛)]<br />

θs = solar zenith angle [°]<br />

The Earth-Sun distance was obtained in astronomical units (d) and solar zenith angle (θs) value from the metadata of the downloaded image. The Recommended Solar Exoatmospheric Spectral Irradiances (ESUN) Values from the USGS website were also included as an aid for the calculations. The final asset of the study was the spatial analyst tool Raster calculator in ArcGIS Pro, which was used to perform the calculation from radiance to ToA reflectance.<br /> 


### Methodology Model of Random Forest Method<br />

#### Normalized Burn Ratio<br />

After preprocessing, each spatiotemporal file (i.e. Nakina 1990, Nakina 2005, Pickle Lake 2020) was classified using the normalized burn ratio (NBR) through raster calculations in ArcGIS Pro software. The NBR index utilizes the near-infrared band (NIR), and shortwave infrared band (SWIR), where the calculation is (NIR + SWIR)/(NIR-SWIR) creating a single raster composition of the NBR. On Landsat 5, the NIR band is band 4, and the SWIR band is band 7. On Landsat 8, the NIR band is band 5, and the SWIR band is band 7. The NBR calculation is a ratio calculation between the NIR and SWIR bands which reveals burned areas, as well as burn severity across a landscape. The resulting raster spreads across a range from -1 to +1, where -1 represents burnt areas, and 1 represents intact land. The NBR files were displayed in the ‘percent clip’ stretch in ArcGIS which highlighted the burnt and unburnt areas, allowing for easier interpretation. <br />

#### Normalized Difference Vegetation Index<br />

After preprocessing, which involves converting the digital number (DN value) to spectral radiance and then transforming radiance to reflectance, the reflectance layer data becomes usable for calculating vegetation indices like the Normalized Difference Vegetation Index (NDVI). It’s crucial to use a raster calculator to eliminate negative reflectance values. The conditional expression is set as Con("band_reflectance" < 0, 0, "band_reflectance"),  which ensures that only reflectance values greater than zero are preserved. NDVI is derived using the formula (NIR - RED) / (NIR + RED), which utilizes the reflectance values from the near-infrared (NIR) and red bands (Rouse et al., 1974). For the Nakina and Pickle Lake datasets from 1990 and 2005, the near-infrared band corresponds to band 4, and the red band to band 3. However, for the 2020 data from both locations, the near-infrared band shifts to band 5, and the red band to band 4.<br />

#### Random Forest Machine Learning Model<br />

The Random Forest (RF) machine learning model utilizes vector training data to make decisions with explanatory rasters. For this study, 1640 training points in Nakina were classified using object-based image analysis (OBIA), ISO cluster classification, NDVI, NBR  and composite images to interpret satellite imagery for the Nakina study site (Son et al., 2015). The seven classes of ‘water’, ‘burnt forest’, ‘regenerated forest’, ‘intact forest’, ‘harvested forest’, ‘logging road’, or ‘peatlands’, were each equally represented across the 1640 training points. Alongside the classes, these features were also given a status of degraded or not degraded.<br />

 For the study, two separate RF models were run - a controlled model on 2020 sites, and a predictive model for 2022 rasters. Both models were trained across all the data in Nakina, but since the RF model is stationary, and doesn’t account for time as a controlled variable,  all 3 time points were included in one model. This included the composite, NBR, and NDVI for 1990, 2005, and 2020. Once the model was trained on this data, we then utilized a mean NDVI and mean NBR so that we could overlap a predictor raster for 2022’s NDVI, and NBR to the historical data, and try to predict the potential degradation of Nakina and Pickle Lake in 2022, which is not a part of the training data. For the Pickle Lake site, the option ‘Compensate for Sparse Categories’ was selected, to account for the potential of having a low presence of degraded points in the unmanaged forest.<br />

## StoryMap<br />
[The classification and application of Random Forest Models](https://storymaps.arcgis.com/stories/ba660050b00a4510a11db7432f90e376)<br />

## Conclusion & Recommendations<br />
In conclusion, the RF model is capable of detecting forest degradation at different rates in managed and unmanaged land in Northern Ontario to show significant differences in forest management practices. The model showed the ability to utilize training data from one context and apply it to a new environment moderately well, however,, it is confirmed by limited input data, and would see further improvement with the inclusion of elevation data, and hydrology mapping to improve decision making. Due to the stationary nature of this model, it would not be of best interest to model time series data with a RF model in the future, and something such as LandTrendr pixel-mapping could be more appropriate for a study of this nature (Kennedy et al., 2010).  With that in mind, this research shows that the RF model can be utilized with open Landsat satellite data to predict areas of forest degradation and that forest degradation is in fact occurring more in managed forests as a result of anthropogenic activity.<br />
