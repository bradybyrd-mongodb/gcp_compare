# ----------------------------------------------------------- #
#                  GCP Compare 
# ----------------------------------------------------------- #
#  A collection of utilities for comparing GCP native databases with mongoDB

### *Principle*

To perform an side-by-side compare we need to be able to generate the same data set in both products.

### *How it works*

All of the data_loader scripts refer to the files in the model-tables folder.  These use a csv format to build both structure and data generation.  Here's a small example:`

`Claim.principalDiagnosis,String,,"str(fake.random_int(min=1, max=50000))",0`
`Claim.priorAuthorization,boolean,,fake.boolean(),0`
`Claim.receivedDate,Date,,fake.past_datetime(),0`
`Claim.claimLine().supervisingProvider_id,String,,"IDGEN.random_value(""P-"")",0`
`Claim.claimLine().unit,String,,"fake.random_element(('Each', 'Days', 'Prints', 'Miles'))",0`
`Claim.claimLine().diagnosiscodes().diagnosisCode,String,,"str(fake.random_int(min=1, max=50000))",0`
`Claim.claimLine().payment.allowedAmount,double,,"fake.random_int(min=0, max=10000)",0`
`Claim.claimLine().payment.approvedAmount,double,,"fake.random_int(min=0, max=10000)",0`

*Using the Mongo loader (relational_replace_loader)* - This would create a single claim document with primary fields (priciple.., prio..., received..), an array of claimLine subdocuments that include a subdocument of payment and an array of diagnosisCodes.

*On the SQL side (load_sql, load_gcp, load_spanner)* -  Each of the hierchical elements would be cast into a separate table.  This will create a claim table, claim_claimLine, claim_claimLine_payment and claim_claimLine_diagnosisCodes.  The scripts automatically add foreign key relationships.

### *Operation*
The scipt is controlled by data in the relations_settings.json file, here you can specify groups of dependent csv templates, each with its own set of ratios.  A typical invocation would be:

`python3 load_sql_spanner.py action=load_data`

This will start the data loader and look for a key in the settings file called "data".  In this case it looks like this:

`"data": {`
   ` "provider": {`
   `   "path": "model-tables/provider.csv",`
   `   "size": 2000,`
   `   "id_prefix": "P-"`
   ` },`
   ` "member": {`
   `   "path": "model-tables/member.csv",`
   `   "size": 10000,`
   `   "id_prefix": "M-",`

It specified a system of csv documents to build a claim-member-provider domain.
An invocation like this:

`python3 load_sql_spanner.py action=load_data template=model-tables/quote.csv`

will just run the quote template following the sizes specifies in settings.
This will perform ddl actions:

`python3 load_sql_spanner.py action=execute_ddl tample=model-tables/quote.csv task=create`

This will create the tables in the quote system.  A task of drop drops, delete deletes etc.

For security, the script looks for environment variables __PWD__ as passwords as well as the OOGLE_APPLICATION_CREDENTIALS to get you into the GCP environment.