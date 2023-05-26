---
title: "Connecting world electricity production with air pollution"
excerpt_separator: "<!--more-->"
categories:
  - Blog
tags:
  - Energy
  - Statistics
  - Health
  - Climate
usemathjax: true
toc: true
toc_sticky: true
---

It is reasonable to assume that particularly dirty energy production, e.g. coal, will be associated with higher death rates from air pollution. This notebook investigates the relationship between different kinds of electricity production and deaths from air pollution using data from Our World in Data. I find that the relationship between electricity production and air pollution deaths is not quite so straightforward. You can also find this notebook, including the code and data, on [GitHub](https://www.github.com/buuck/world_electricity_and_air_pollution).


### Load and prepare data

I have two shapefile sets, but they both have issues. The One is old and is missing at least one new country (South Sudan). The one in `world_countries_generalized` is better, but for some reason it does not separate China from Taiwan... So I'll combine them by hand by loading the geometries for China and Taiwan from `world_shapefiles` into `world_countries_generalized`.

Here is a sample from `world_countries_generalized`:
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
      <th>FID</th>
      <th>NAME</th>
      <th>ISO</th>
      <th>COUNTRYAFF</th>
      <th>AFF_ISO</th>
      <th>SHAPE_Leng</th>
      <th>SHAPE_Area</th>
      <th>geometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>73</th>
      <td>74</td>
      <td>Ethiopia</td>
      <td>ET</td>
      <td>Ethiopia</td>
      <td>ET</td>
      <td>46.810315</td>
      <td>92.722761</td>
      <td>POLYGON ((45.48940 5.48976, 45.37446 5.36392, ...</td>
    </tr>
    <tr>
      <th>247</th>
      <td>248</td>
      <td>Wallis and Futuna</td>
      <td>WF</td>
      <td>France</td>
      <td>FR</td>
      <td>0.700608</td>
      <td>0.013414</td>
      <td>MULTIPOLYGON (((-178.06082 -14.32389, -178.137...</td>
    </tr>
    <tr>
      <th>56</th>
      <td>57</td>
      <td>Côte d'Ivoire</td>
      <td>CI</td>
      <td>Côte d'Ivoire</td>
      <td>CI</td>
      <td>31.576752</td>
      <td>26.340497</td>
      <td>MULTIPOLYGON (((-5.33971 5.19775, -5.31977 5.1...</td>
    </tr>
    <tr>
      <th>10</th>
      <td>11</td>
      <td>Armenia</td>
      <td>AM</td>
      <td>Armenia</td>
      <td>AM</td>
      <td>12.161117</td>
      <td>3.142291</td>
      <td>MULTIPOLYGON (((46.54037 38.87559, 46.51639 38...</td>
    </tr>
    <tr>
      <th>246</th>
      <td>247</td>
      <td>Vietnam</td>
      <td>VN</td>
      <td>Viet Nam</td>
      <td>VN</td>
      <td>66.866802</td>
      <td>27.556082</td>
      <td>MULTIPOLYGON (((107.07896 17.10804, 107.08333 ...</td>
    </tr>
    <tr>
      <th>228</th>
      <td>229</td>
      <td>Trinidad and Tobago</td>
      <td>TT</td>
      <td>Trinidad and Tobago</td>
      <td>TT</td>
      <td>4.384972</td>
      <td>0.413753</td>
      <td>MULTIPOLYGON (((-61.07945 10.82416, -61.07556 ...</td>
    </tr>
    <tr>
      <th>69</th>
      <td>70</td>
      <td>Equatorial Guinea</td>
      <td>GQ</td>
      <td>Equatorial Guinea</td>
      <td>GQ</td>
      <td>8.191007</td>
      <td>2.188207</td>
      <td>MULTIPOLYGON (((10.41505 1.00250, 10.30861 1.0...</td>
    </tr>
    <tr>
      <th>40</th>
      <td>41</td>
      <td>Cameroon</td>
      <td>CM</td>
      <td>Cameroon</td>
      <td>CM</td>
      <td>41.960596</td>
      <td>37.972713</td>
      <td>POLYGON ((10.18107 2.16786, 10.07389 2.16778, ...</td>
    </tr>
    <tr>
      <th>162</th>
      <td>163</td>
      <td>Niue</td>
      <td>NU</td>
      <td>New Zealand</td>
      <td>NZ</td>
      <td>0.541413</td>
      <td>0.021414</td>
      <td>POLYGON ((-169.89389 -19.14556, -169.93088 -19...</td>
    </tr>
    <tr>
      <th>34</th>
      <td>35</td>
      <td>Brunei Darussalam</td>
      <td>BN</td>
      <td>Brunei Darussalam</td>
      <td>BN</td>
      <td>4.918828</td>
      <td>0.468299</td>
      <td>MULTIPOLYGON (((115.01844 4.89579, 114.98915 4...</td>
    </tr>
  </tbody>
</table>
</div>


Here is a sample of `world_shapefiles`:


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
      <th>NAME</th>
      <th>geometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>203</th>
      <td>United Republic of Tanzania</td>
      <td>MULTIPOLYGON (((39.68250 -7.99333, 39.65305 -7...</td>
    </tr>
    <tr>
      <th>17</th>
      <td>Burma</td>
      <td>MULTIPOLYGON (((98.03581 9.78639, 98.03027 9.7...</td>
    </tr>
    <tr>
      <th>59</th>
      <td>Finland</td>
      <td>MULTIPOLYGON (((23.70583 59.92722, 23.64944 59...</td>
    </tr>
    <tr>
      <th>166</th>
      <td>Guinea-Bissau</td>
      <td>MULTIPOLYGON (((-15.88583 11.05222, -15.92556 ...</td>
    </tr>
    <tr>
      <th>225</th>
      <td>Netherlands Antilles</td>
      <td>MULTIPOLYGON (((-68.19528 12.22111, -68.19278 ...</td>
    </tr>
    <tr>
      <th>214</th>
      <td>Viet Nam</td>
      <td>MULTIPOLYGON (((106.60027 8.64778, 106.59248 8...</td>
    </tr>
    <tr>
      <th>60</th>
      <td>Fiji</td>
      <td>MULTIPOLYGON (((-178.70776 -20.67444, -178.715...</td>
    </tr>
    <tr>
      <th>21</th>
      <td>Bulgaria</td>
      <td>POLYGON ((27.87917 42.84110, 27.89500 42.80250...</td>
    </tr>
    <tr>
      <th>6</th>
      <td>American Samoa</td>
      <td>MULTIPOLYGON (((-170.54251 -14.29750, -170.546...</td>
    </tr>
    <tr>
      <th>72</th>
      <td>Guam</td>
      <td>POLYGON ((144.70941 13.23500, 144.70245 13.235...</td>
    </tr>
  </tbody>
</table>
</div>


The data table containing the death types is quite large and contains many different kinds of deaths. For now we are only going to look at Outdoor Air Pollution deaths.


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
      <th>NAME</th>
      <th>Code</th>
      <th>Year</th>
      <th>Deaths - Cause: All causes - Risk: Outdoor air pollution - OWID - Sex: Both - Age: All Ages (Number)</th>
      <th>Deaths - Cause: All causes - Risk: High systolic blood pressure - Sex: Both - Age: All Ages (Number)</th>
      <th>Deaths - Cause: All causes - Risk: Diet high in sodium - Sex: Both - Age: All Ages (Number)</th>
      <th>Deaths - Cause: All causes - Risk: Diet low in whole grains - Sex: Both - Age: All Ages (Number)</th>
      <th>Deaths - Cause: All causes - Risk: Alcohol use - Sex: Both - Age: All Ages (Number)</th>
      <th>Deaths - Cause: All causes - Risk: Diet low in fruits - Sex: Both - Age: All Ages (Number)</th>
      <th>Deaths - Cause: All causes - Risk: Unsafe water source - Sex: Both - Age: All Ages (Number)</th>
      <th>...</th>
      <th>Deaths - Cause: All causes - Risk: High body-mass index - Sex: Both - Age: All Ages (Number)</th>
      <th>Deaths - Cause: All causes - Risk: Unsafe sanitation - Sex: Both - Age: All Ages (Number)</th>
      <th>Deaths - Cause: All causes - Risk: No access to handwashing facility - Sex: Both - Age: All Ages (Number)</th>
      <th>Deaths - Cause: All causes - Risk: Drug use - Sex: Both - Age: All Ages (Number)</th>
      <th>Deaths - Cause: All causes - Risk: Low bone mineral density - Sex: Both - Age: All Ages (Number)</th>
      <th>Deaths - Cause: All causes - Risk: Vitamin A deficiency - Sex: Both - Age: All Ages (Number)</th>
      <th>Deaths - Cause: All causes - Risk: Child stunting - Sex: Both - Age: All Ages (Number)</th>
      <th>Deaths - Cause: All causes - Risk: Discontinued breastfeeding - Sex: Both - Age: All Ages (Number)</th>
      <th>Deaths - Cause: All causes - Risk: Non-exclusive breastfeeding - Sex: Both - Age: All Ages (Number)</th>
      <th>Deaths - Cause: All causes - Risk: Iron deficiency - Sex: Both - Age: All Ages (Number)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Afghanistan</td>
      <td>AFG</td>
      <td>1990</td>
      <td>3169</td>
      <td>25633</td>
      <td>1045</td>
      <td>7077</td>
      <td>356</td>
      <td>3185</td>
      <td>3702</td>
      <td>...</td>
      <td>9518</td>
      <td>2798</td>
      <td>4825</td>
      <td>174</td>
      <td>389</td>
      <td>2016</td>
      <td>7686</td>
      <td>107</td>
      <td>2216</td>
      <td>564</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Afghanistan</td>
      <td>AFG</td>
      <td>1991</td>
      <td>3222</td>
      <td>25872</td>
      <td>1055</td>
      <td>7149</td>
      <td>364</td>
      <td>3248</td>
      <td>4309</td>
      <td>...</td>
      <td>9489</td>
      <td>3254</td>
      <td>5127</td>
      <td>188</td>
      <td>389</td>
      <td>2056</td>
      <td>7886</td>
      <td>121</td>
      <td>2501</td>
      <td>611</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Afghanistan</td>
      <td>AFG</td>
      <td>1992</td>
      <td>3395</td>
      <td>26309</td>
      <td>1075</td>
      <td>7297</td>
      <td>376</td>
      <td>3351</td>
      <td>5356</td>
      <td>...</td>
      <td>9528</td>
      <td>4042</td>
      <td>5889</td>
      <td>211</td>
      <td>393</td>
      <td>2100</td>
      <td>8568</td>
      <td>150</td>
      <td>3053</td>
      <td>700</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Afghanistan</td>
      <td>AFG</td>
      <td>1993</td>
      <td>3623</td>
      <td>26961</td>
      <td>1103</td>
      <td>7499</td>
      <td>389</td>
      <td>3480</td>
      <td>7152</td>
      <td>...</td>
      <td>9611</td>
      <td>5392</td>
      <td>7007</td>
      <td>232</td>
      <td>411</td>
      <td>2316</td>
      <td>9875</td>
      <td>204</td>
      <td>3726</td>
      <td>773</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Afghanistan</td>
      <td>AFG</td>
      <td>1994</td>
      <td>3788</td>
      <td>27658</td>
      <td>1134</td>
      <td>7698</td>
      <td>399</td>
      <td>3610</td>
      <td>7192</td>
      <td>...</td>
      <td>9675</td>
      <td>5418</td>
      <td>7421</td>
      <td>247</td>
      <td>413</td>
      <td>2665</td>
      <td>11031</td>
      <td>204</td>
      <td>3833</td>
      <td>812</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 31 columns</p>
</div>



There are a bunch of missing values in the electricity production data table. For some countries, like Afghanistan, there does not appear to be electricity production data dating all the way back to 1900, which is fine. We will simply ignore those periods of time for countries that do not have them.



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
      <th>NAME</th>
      <th>Year</th>
      <th>Code</th>
      <th>population</th>
      <th>gdp</th>
      <th>biofuel_cons_change_pct</th>
      <th>biofuel_cons_change_twh</th>
      <th>biofuel_cons_per_capita</th>
      <th>biofuel_consumption</th>
      <th>biofuel_elec_per_capita</th>
      <th>...</th>
      <th>solar_share_elec</th>
      <th>solar_share_energy</th>
      <th>wind_cons_change_pct</th>
      <th>wind_cons_change_twh</th>
      <th>wind_consumption</th>
      <th>wind_elec_per_capita</th>
      <th>wind_electricity</th>
      <th>wind_energy_per_capita</th>
      <th>wind_share_elec</th>
      <th>wind_share_energy</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Afghanistan</td>
      <td>1900</td>
      <td>AFG</td>
      <td>4832414.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Afghanistan</td>
      <td>1901</td>
      <td>AFG</td>
      <td>4879685.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Afghanistan</td>
      <td>1902</td>
      <td>AFG</td>
      <td>4935122.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Afghanistan</td>
      <td>1903</td>
      <td>AFG</td>
      <td>4998861.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Afghanistan</td>
      <td>1904</td>
      <td>AFG</td>
      <td>5063419.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 129 columns</p>
</div>



### Analysis of electricity production and air pollution deaths
Now we are ready to analyze this data set.

This plot shows the per-capita electricity use by country in 2019. It mostly tracks how you might expect, with less wealthy countries using less electricity. There are some outliers, like Iceland. This is apparently because there is a very strong aluminum industry in Iceland, which uses a lot of electricity. That, combined with Iceland's very small population, leads to very high per-capita electricity usage.

    
![png]({{ "/assets/images/world_electricity_and_air_pollution_files/world_electricity_and_air_pollution_19_0.png" | relative_url }})
    


We can also see how much coal each country burns to generate electricity on a per capita basis. Australia is a clear outlier with a handful of other countries (China, the United States, South Africa, Kazakhstan, and some Eastern/Southeastern European countries) in a second usage tier.

    
![png]({{ "/assets/images/world_electricity_and_air_pollution_files/world_electricity_and_air_pollution_21_0.png" | relative_url }})
    


Next let's look at annual deaths from outdoor air pollution per capita. There is some correlation between this plot and the previous one showing coal usage for electricity per capita (compare China and Eastern Europe), but there are plenty of places where outdoor air pollution is bad even though there is not much coal being used (Africa, the Middle East, etc.).

    
![png]({{ "/assets/images/world_electricity_and_air_pollution_files/world_electricity_and_air_pollution_23_0.png" | relative_url }})


Let's make some more plots to investigate the degree to which deaths from outdoor air pollution are related to fossil fuel use. Because the total number of countries in the world is large, we will focus on analyzing data from the 6 most populous and richest countries. These are nearly mutually exclusive sets which is nice for comparative purposes. There are several very wealthy but very small countries with unusual economies that are not particularly representative (e.g. Qatar, Luxembourg, Singapore), so I am excluding any country with a population less than 10 million. I am also excluding Saudi Arabia becauase, while its population is greater than 10 million, as an oil exporter its electricity source mix is unusually skewed towards oil.

    Six most populous countries in the world:
    
    China
    India
    United States
    Indonesia
    Pakistan
    Brazil
    
    Six richest countries in the world (GDP per capita) with a population
      greater than 10 million people (excl. Saudi Arabia):
    
    United States
    Australia
    Netherlands
    Germany
    Sweden
    Canada

Below we see a series of plots comparing the annual number of deaths per capita from outdoor air pollution to a variety of metrics, mostly related to electricity production. However, before we start looking at those comparisons, I want to show this plot that motivates what I said above: that the most populous countries are generally not the wealthiest countries. They basically fall into three categories, with China, Indonesia, and Brazil having achieved essentially middle-income status, while India and Pakistan are still relatively poor (but getting richer quickly, particularly India!) and the US is much wealthier than the rest. Among the rich countries, we see that they are all pretty much the same, with the US (and to a lesser extent Australia) being a bit wealthier than the rest.

It's also instructive to look at the scales of the x-axes. The wealthy countries are all within ~20% of each other in GDP per capita, while Brazil, Indonesia, and China are all more than twice as wealthy as India and Pakistan. 
    
![png]({{ "/assets/images/world_electricity_and_air_pollution_files/world_electricity_and_air_pollution_32_0.png" | relative_url }})
    


Now we'll look at the relationships that outdoor air pollution deaths have with different electricity production metrics.

The first plot compares to the annual amount of electricity generated per capita from coal. One might expect that this would be correlated with deaths from outdoor air pollution, and ideed we see that this is the case for both populous and rich countries. The lightness of color for each country shows the progression over time, with later years being darker. We can see that, apart from the US, the most populous countries in the world have mostly seen increasing deaths from outdoor air pollution over time, which mostly corresponds to an increase in electricity generation from coal. The rich countries are doing the opposite: reducing their coal electricity generation and simultaneously reducing their rates of outdoor air pollution deaths. The Netherlands are somewhat of an outlier among the rich countries, as they seem to be increasing their coal electricity generation over time, but this has not yet lead to an increase in air pollution deaths. It turns out that there are factors besides coal electricity generation that affect air pollution deaths!

    
![png]({{ "/assets/images/world_electricity_and_air_pollution_files/world_electricity_and_air_pollution_34_0.png" | relative_url }})
    


The second plot make the same comparison with solar electricity generation per capita. Here we see that both groups of countries are deploying more solar power, but this is of course not correlated with a decrease in deaths from outdoor air pollution in the populous countries. Building out renewables is not enough; we need to reduce emissions!
    
![png]({{ "/assets/images/world_electricity_and_air_pollution_files/world_electricity_and_air_pollution_36_0.png" | relative_url }})
    


If we compare to the fraction of electricity that is generated by fossil fuels, we see somewhat different relationships. I'll note that these plots have a logisticly scaled x-axis to better show the behavior of countries with either very high or very low fractions of fossil fuel electricity generation.

Even though China is increasing its coal usage for electricity, it is simultaneously reducing the fraction of its electricity that comes from fossil fuels. This is because it is building renewables even faster! We see that most of the other populous countries are mostly holding steady in fossil fuel use, with the exceptions of the US and Brazil. Brazil is unfortunately increasing its fossil fuel use, although it is important to note that this is from an incredibly low baseline.

The rich countries are all reducing their fossil fuel use overall, but there is wide variation in where they are in that process. Sweden has a nearly completely clean grid, while the Netherlands and Australia have comparitvely dirty grids.
![png]({{ "/assets/images/world_electricity_and_air_pollution_files/world_electricity_and_air_pollution_38_0.png" | relative_url }})
    


Finally let's compare to the fraction of electricity generated from renewable energy sources. Again, I plot the x-axis on a logistic scale. These are basically just inverses of the previous plots. We can see that the Netherlands and Germany have really scaled their renewable energy over the past 30 years or so, while other countries have been slower.
    
![png]({{ "/assets/images/world_electricity_and_air_pollution_files/world_electricity_and_air_pollution_40_0.png" | relative_url }})
    


### Extracting some numbers from the above plots

Some of the relationships in the above plots look vaguely linear, so let's see if we can successfully apply that hypothesis to the rest of our data set. First we'll take a look at the relationship between the share of electricity generated from fossil fuels and the number of deaths from outdoor air pollution per capita. Because the plot we made showing this relationship had log-logistic axes, the relationship in linear space is complicated. Suffice it to say that a positive slope (in the log-logistic space) indicates that deaths from outdoor air pollution generally go up when the fraction of electricty from fossil fuels increases, and a negative slope indicates the reverse. I plot only those countries that had sufficient data to perform a fit that returned a nonzero slope with a p-value less than 0.1. Countries that do not meet these criteria are shown in gray. I'm using a relatively high p-value because this is not a particularly rigorious study and we are more interested in seeing possible relationships than quantifying anything.

    
![png]({{ "/assets/images/world_electricity_and_air_pollution_files/world_electricity_and_air_pollution_45_0.png" | relative_url }})
    


In my opinion, the above figure shows that, in general, weathier countries see a positive relationship between fossil fuel electricity share and deaths from outdoor air pollution. I.e. deaths decrease as fossil fuel use decreases. In particular, I am looking at Europe, North America, and Australia/New Zealand. There are a few outliers in that group, namely Norway, Latvia, Switzerland, Serbia, and North Macedonia. Conversely, many countries in the developing world have a negative relationship between these two variables. I suspect that this is probably related to the fact that as they develop, they are simultaneously increasing fossil fuel usage and improving their public health and healthcare so that illnesses from outdoor air pollution are less likely to be fatal.

Let's see to what degree this relationship between this slope and a country's wealth holds up. Below I plot these slopes against each country's GDP per capita.

    
![png]({{ "/assets/images/world_electricity_and_air_pollution_files/world_electricity_and_air_pollution_47_0.png" | relative_url }})
    


We can see that there is some relationship between the two, primarily for countries with a GDP per capita above ~$30,000 per year.

Now let's look at the relationship between each country's annual electricity generation from coal and the number of deaths from outdoor air pollution per capita. Again, I plot only those countries that had sufficient data to perform a fit that returned a nonzero slope with a p-value less than 0.1. Countries that do not meet these criteria are shown in gray.
    
![png]({{ "/assets/images/world_electricity_and_air_pollution_files/world_electricity_and_air_pollution_51_0.png" | relative_url }})
    


Here we see that most countries have a positive relationship between coal electricity generation and deaths from outdoor air pollution. Norway is clearly an outlier, and several countries with apparently negative power-law exponents are also outliers. Below I show a plot comparing some of those countries.

   
![png]({{ "/assets/images/world_electricity_and_air_pollution_files/world_electricity_and_air_pollution_53_0.png" | relative_url }})
    


We see that Norway is an outlier because it has had very little coal in its electricity generation mix for the entirety of this data set. That, combined with its great strides in reducing deaths from air pollution, mean that the slope of this relationship is very large for Norway. Italy and Kazakhstan are also outliers on this plot in that they have negative slopes. For Italy we can see that, even though its fitted slope apparently has a p-value < 0.1, its evolution over time does not seem to be particularly linear. Kazakhstan does seem to be reducing air pollution deaths while increasing its coal usage, perhaps because it is still a developing country.
