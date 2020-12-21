# Streaming Time Series with SPSS Flow Modeler Hands-on Lab
(C) IBM 2019 - Ecosystem Advocacy Group Europe

Author: philippe.gregoire@fr.ibm.com

Reference Material:
* https://www.ibm.com/support/knowledgecenter/en/SS3RA7_sub/modeler_mainhelp_client_ddita/clementine/timeser_as_node_general.html

* https://www.ibm.com/support/knowledgecenter/en/SS3RA7_16.0.0/com.ibm.spss.modeler.help/clementine/streamingts_deploymenttab.htm#streamingts_deploymenttab

## Setup
Create a project in Watson Studio, named e.g. `StreamingTS`
![](Lab5_StreamingTimeSeries.assets/Lab5_StreamingTimeSeries-f7d1847a.png)

## Add input file as asset
Upload `broadband.csv` as file Data Asset in your project: ![](Lab5_StreamingTimeSeries.assets/Lab5_StreamingTimeSeries-90566d7a.png)

## Create SPSS Flow with Modeler
### Setup flow with asset as input
1. Create new Modeler Flow using the `[(+) Add to project]` button: ![](images_Lab4/20190214_f51e8d61.png)
1. Give it a name, e.g. *SPSS Streaming TS*, making sure the type is *Modeler Flow* and runtime *IBM SPSS Modeler* ![](images_Lab4/20190214_ef06293f.png)
3. One the flow canvas is displayed, from the `Import` palette drawer, add a `Data Asset` node: ![](images_Lab4/20190214_38973598.png)
4. Open the node from its menu: ![](images_Lab4/20190214_9235522e.png)
5. Set the Data Asset to the `broadband.csv` file ![](images_Lab4/20190214_5d96dcef.png)
6. From the `Outputs` palette, drop a `Data Audit` node an wire it to the Data Asset node: ![](images_Lab4/20190214_119a1615.png)
7. Run the flow using the `Play` icon: ![](images_Lab4/20190214_2fd46f9e.png)
8. Switch to the View output tab on the right panel: ![](images_Lab4/20190214_094004bf.png)
9. Open the Data Audit ![](images_Lab4/20190214_8c5d07cb.png)
10. We get a summary of the data in the set, basically `Market_1` to `Market_85` columns with 60 rows each, a `DATE_` field

### Convert data into usable form
We will want to work on a subset of the markets, and use the `DATE_` column as the time indicator.
To get an understanding of the fields, we'll visualize them as a time plot:
1. From the `Graphs` drawer, select a `Time plot` node and wire it to the Data Asset node: ![](images_Lab4/20190214_3cdee63a.png)
1. Open the `Time Plot` node ![](images_Lab4/20190214_e944d710.png)
1. Select the `DATE_` as custom X axis, uncheck *separate panel* and *normalize*: ![](images_Lab4/20190214_76f6dce4.png)
1. Add all the `Market_*` columns: ![](images_Lab4/20190214_b3fac619.png) except `Total`, `YEAR_`, `MONTH_` and `DATE_` (Select all from the upper check box, then unselect the unwanted ones)   
![](images_Lab4/20190214_289f5a05.png)
1. Now Run the graph node: ![](images_Lab4/20190214_af6dc4b0.png)
1. Open the graph ![](images_Lab4/20190214_6b25cb47.png)
1. This gives an idea of how the markets behave, all are more or less increasing, with various amplitudes. ![](images_Lab4/20190214_b1c99040.png) To visualize relative volatility, you may want to re-run the graph node with `Normalize` checked.
9. In the filter node, select to retain fields, and keep only `Market_1` to `Market_5` and `DATE_`: ![](images_Lab4/20190214_91b03232.png)
1. For the purpose of the lab, we will subselect only the 5 first markets. From `Field Operations` drop a `Filter` node and wire it to the Data Asset: ![](images_Lab4/20190214_4cc1c08a.png)
1. Add a `Filler` node and wire it after the `Filter` node. Setup the `DATE_` column to fill-in, with condition `Always` ![](images_Lab4/20190214_c1351ecb.png)
1. Set the `Replace with` formula to `to_date('DATE_')`: ![](images_Lab4/20190214_2bd3ee85.png)
1. We will need to specify which fields are to be used as input or output. Add a *Type* node and wire it after the *Filler* node: ![](images_Lab4/20190215_fea3488b.png)

## Build the Streaming TimeSeries
1. Setup the *Type* node so that the `Market_*` fields have a role of `Both`, while the `DATE_` field has a role of `Input`: ![](images_Lab4/20190215_5edc1de2.png)
1. From `Record Operations` add a `Streaming TS` ![](images_Lab4/20190214_f999cc1d.png) node and wire it after the *Type* node.
1. Setup the *Streaming TS* node, add the `Market_*` fields both as *Targets* and *Candidate Input*:![](images_Lab4/20190215_233b2ee8.png)
1. In *OBSERVATIONS AND TIME INTERVAL*, setup `DATE_` as Date/Time field, `Months` for time interval, `1` for the increment.
1. In *MODEL OPTIONS*, setup *Expert Modeler*, for all models, and select both *Seasonal* and *Sophisticated Exponential Smoothing* methods: ![](images_Lab4/20190215_0cfef086.png)
1. Then setup forecast to `5` records in the future, and select confidence and residuals computation: ![](images_Lab4/20190215_4740eca2.png)
1. Add another *Time Plot* node from the `Graphs` drawer and wire it after the Streaming node. Setup the node so that it displays the `Market_1` columns. You may need to run the node once to materialize the colums. We will get additional columns for the predictions:
   * `$TS-Market_1`: the sliding window prediction
   * `$TSLCI-Market_1` and `$TSUCI-Market_1`: the Lower and Upper Confidence Indicators for the prediction.
     * `$TSResidual-Market_1`: the computed residuals of prediction vs actuals
1. Add the `DATE_` as X axis label: ![](images_Lab4/20190215_9fd7ee01.png)
1. Run the graph and then display it:
1. You may want to do the same for e.g. `Market_2` field

## Deploy Streaming TimeSeries model
1. In order to get the output of the deployment, add a `Table` node from the *Outputs* drawer, and wire it to the *Streaming TS* node: ![](images_Lab4/20190215_cfbc6d94.png)
1. From the *Table* node menu, select `Save branch as model`: ![](images_Lab4/20190215_e1680e0d.png).
1. In the *Save Model* panel, make sure the `Table` branch is selected as terminal node, give it a name, e.g. `Markets_Predictions` and save: ![](Lab5_StreamingTimeSeries.assets/Lab5_StreamingTimeSeries-1afdad69.png)
1. We will now promote the model to a deployment space from where we can expose through a REST API endpoint. Switch back to your project, locate you newly saved model in the *Models* section and select *Promote* from its menu: ![](Lab5_StreamingTimeSeries.assets/Lab5_StreamingTimeSeries-92d26f0b.png)
1. Select the `[New space +]` button ![](Lab5_StreamingTimeSeries.assets/Lab5_StreamingTimeSeries-76cf65d5.png)
1. Give the deployment space a name, e.g. `Market_DeplSpace`. Make sure your instance of *Watson Machine Learning* is selected and click `[Create]` ![](Lab5_StreamingTimeSeries.assets/Lab5_StreamingTimeSeries-2a789499.png), wait for it to be ready and close.
1. Back in the Promote dialog, select your new space as target space and click `[Promote]` ![](Lab5_StreamingTimeSeries.assets/Lab5_StreamingTimeSeries-d16c5fc5.png)
1. From the top-left hamburger menu, select *View all spaces* ![](Lab5_StreamingTimeSeries.assets/Lab5_StreamingTimeSeries-e8aa0198.png)
1. Open your `Market_DeplSpace` ![](Lab5_StreamingTimeSeries.assets/Lab5_StreamingTimeSeries-ae90beb3.png)
1. Click on the rocket button to deploy ![](Lab5_StreamingTimeSeries.assets/Lab5_StreamingTimeSeries-7ebb669d.png)
1. Select Online type tile, give it a name, e.g. `Online_Market_Predict`, and click `[Create]`: ![](Lab5_StreamingTimeSeries.assets/Lab5_StreamingTimeSeries-2ed368eb.png)
1. Switch to the Deployments tab and open your deployment once it will be in `Deployed` state ![](Lab5_StreamingTimeSeries.assets/Lab5_StreamingTimeSeries-b1befd96.png)
1. Copy the Direct link Endpoint value to clipboard ![](Lab5_StreamingTimeSeries.assets/Lab5_StreamingTimeSeries-b67f09e7.png)

## Invoke the REST endpoint from a notebook
We could test the deployed service from the Cloud UI interface, but the volume of input data does not make it very practical.   
So, we will use a Python Jupyter notebook to test and display predictions.

1. The test notebook has been prepared for you. It is derived from the Python implementation sample code, but is using the WML Python wrapper library to find the model named `Markets_Predictions`. The WML python client library is documented at https://wml-api-pyclient.mybluemix.net/
1. Switch back to your project and add an asset of type *Notebook*: ![](images_Lab4/20190215_57058974.png)
1. Switch to the *From file* tab, and choose  `WML_StreamingTS_Predictions.ipynb` notebook, select the 'Default Python 3.5 Free' environment: ![](images_Lab4/20190215_c7fc03df.png)
1. In the notebook, you will need to specify your own Cloud Object Storage (COS) and IAM credentials.
1. Locate the cell which contains `cos_credentials = {`, open the data asset panel on the right, and from the `broadband.csv` file, select `Insert to code/Insert Credentials` to override the existing assignement. Make sure the variable name is `cos_credentials` and not `credentials_1`: ![](images_Lab4/20190215_b20a0dfd.png)
1. Locate the cell which contains `IAM_APIKEY=`. To generate an API_KEY, navigate to https://cloud.ibm.com/iam/apikeys in a separate browser tab.
1. Follow the notebook execution flow, which will drive predictions and chart the results.
