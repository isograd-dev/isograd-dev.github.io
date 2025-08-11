---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults
layout: home
---

<style>
  
</style>

<div id="toc">

* TOC
{:toc}

</div>

<div id="content">

# Introduction
The aim of this document is to describe how to use the services the Isograd system offers, either to LTI consumers or via the API.



# Quick start
## LTI
### Connexion

Isograd plateform is compatible with both LTI 1.1 and LTI 1.3 protocols

#### LTI 1.1
Your system should include an area to define LTI providers. In this area, you must:
- Give a name to Isograd LTI Provider
- Specify your client key
- Specify your client secret
Your client key and your client secret will be provided by Isograd upon request.


#### LTI 1.3

**What we require from you**

![](/assets/img/platform.png)

**What we will provide to you**

![](/assets/img/tool.png)


### Environments
There are two environments, one for testing and the other for production:
- Testing: https://recette.isograd.com/public/lti.php 
- Production : https://app.isograd.com/public/lti.php


### Use a service
Most of the calls made to Isograd LTI Services require **custom parameters** to indicate what you want to achieve. Your system should include the possibility to add custom parameters in your LTI messages.

![](/assets/img/lti_additional_param.jpg)


### Example
To manually test your requests, you can use the LTI Emulation tool accessible here: [https://saltire.lti.app/platform](https://saltire.lti.app/platform). Below is an example of such a test:
![](/assets/img/lti_message.jpg)
![](/assets/img/lti_user.jpg)



## API

### Connexion
The authentication is done via [OAuth](https://oauth.net/getting-started/). To get a token, you need to make a POST request to the following URL:
```
https://auth.isograd.com/oauth2/token?grant_type=client_credentials&scopes=usage
```
This call must contain two fields: `client_id` and `client_secret`. The content of these fields must be the two values Isograd's team provided you with once your account is created.

### Environments
There are two environments, one for testing and the other for production:
- Testing: https://recette.isograd.com/api/usage
- Production : https://app.isograd.com/api/usage


### Use a service
To use a service, make a POST request to the URL of the chosen environment. The header must contain the field `Authorization: Bearer <your_token>`.

### Example
Here is a Python script retrieving an access token, and using the service to [create a candidate](#create-a-candidate)

```python
import requests

# Retrieve the access token
credentials = {
    'client_id': 's0u2c72bman19nm2c45oi1fpea',
    'client_secret': '1mio0273po6si9tvt2f3ld5t6o4hsg02epq71rlfdhlkombkv8ro',
}
auth_url = 'https://auth.isograd.com/oauth2/token?grant_type=client_credentials&scopes=usage'

r_token = requests.post(auth_url, data=credentials)
access_token = r_token.json()['access_token']

# Use the service
headers = {'Authorization': 'Bearer ' + access_token}
url_dev = 'https://recette.isograd.com/api/usage'

payload = {
    'act_id': 1,
    'gen_id': 3,
    'ema': 'test@isograd.com',
    'fst_nam': 'John',
    'lst_nam': 'Doe',
}

r_action = requests.post(url_dev, data=payload, headers=headers)
print(r_action.json())

```

The returned JSON object is like so:
```js
{
    'error_message': null,
    'success': true,
    'can_id': 1381134,
    'lgn_url': 'https://recette.isograd.com/continuelogin/CandidateContinueLogin?param=UVR1WjgwRXBwU0'
}
```


# Actions
All your actions must contain a parameter `act_id` that contains the action you want to do: these actions and the parameters that must be sent are described in this section. For every parameter, there will be an indication of whether or not it is required:
- 🟩 Required parameter
- 🔷 **API**: Required parameter **LTI**: parameter already sent via the LTI protocol, do not include it in the additional parameters
- 🟠 Optional parameter 

The answer will be a JSON object, containing a boolean `success`: if `true`, additional properties are sent depending on the action. If `false`, an `error_code` and an `error_message` will be included. Here are the general error messages  (additional codes specific to each actions are provided later in this document)

| error_code  | error_message  |
|---|---|
| 103 | The property act_id is not valid |
| 105 | Property "{name of property}" is requested  |


## Candidate and tests
### Create a candidate

| Parameter  | Required  | Value |
|---|---|---|
| act_id | 🟩 | 1 |
| gen_id | 🟩  | 1: male, 2: female, 3: not provided |
| ema | 🔷 | Candidate's email address |
| fst_nam | 🔷 | Candidate's first name |
| lst_nam | 🔷 | Candidate's last name |
| lan_id | 🟠 | Language code. See the [appendix](#language-codes) |
| ext_id | 🟠 | Your internal id for the candidate. In LTI, the LTI `UserID` is stored if no `ext_id` is provided |
| dep_id | 🟠 | The id of a Group within Isograd system to which the candidate should be added. Must be empty if a `ext_dep_id` value is provided|
| ext_dep_id | 🟠 | The id of a group within your system to which the candidate should be added. If no group with this id exists, it will be automatically created |
| psw | 🟠 | The candidate's password. If not provided, the system will generate one randomly |
| no_psw_rst | 🟠 | If set to 1 the password will be hashed, it will not be displayed in registration emails and the candidate will not have to change it upon first login |
| job_id | 🟠 | The id of the candidate's job profile |
| xtr_tim_fac | 🟠 | Percentage factor defining the additional time granted to a candidate (e.g., 0 for no extra time, 50 for +50% time, 100 for +100% time, 200 for +200% time), applied to both total test duration and in-application question time |


The response is a JSON object containing the following properties:
- `can_id`: unique integer id of the created candidate on Isograd's platform
- `lgn_url`: a url that can be used by the candidate to login into the platform without entering any credentials. This url is only valid for 30 minutes

**Errors**

| error_code  | error_message  |
|---|---|
| 201 | The candidate could not be created (probably because one of the parameters is not valid) |
| 202 | There is already a candidate with this email on your account. In such case, the response includes a `can_id` property with the id of the corresponding candidate. |
| 203 | Couldn’t find the mentioned `dep_id` in the account and can’t create candidate |

### Add a test to a candidate

| Parameter  | Required  | Value |
|---|---|---|
| act_id | 🟩 | 2: do not add the test if the candidate has an unfinished test for this `tst_frm_id`. <br />9: Create the test even if the candidate has an unfinished test. |
| tst_frm_id | 🟩  | The test identifier. To get the list, log into the Isograd's platform and click on "Help" in the left menu. For compatibility reasons this parameter may be named rea_tst_id until 31/12/2025. |
| ema | 🔷 | Candidate's email address |
| nee_ful_scr | 🟠 | 1: Require full screen for the test |
| html | 🟠 | 1: The response will be an HTML `<a>` tag (with a class attribute `isograd_start_test_button`) |
| redirect | 🟠 | 1: The response will be a HTTP 302 status redirecting to the URL of the test start page |
| ses_id | 🟠 | The session ID to which the test has to be associated |
| add_pro | 🟠 | 1: add remote proctoring to the test. Additional cost will apply.  |
| max_num_tst | 🟠 | The maximum numbers of tests with this `tst_frm_id` the candidate is allowed to take |
| max_num_tst_per | 🟠 | The maximum numbers of tests with this `tst_frm_id` the candidate is allowed to take every X day(s)|
| rtn_pag | 🟠 | The URL of the page to which candidates will be redirected after submitting their feedback (or their results if they are allowed to see them) |
| cal_bac_pag | 🟠 | A URL to which the plateform will submit a GET request when the test is complete(before displaying the feedbacks/results page) |
| cpf_id | 🟠 | The ID of the "Compte Personnel de Formation" file associated to this test |
| nb_test_by_credit | 🟠 | Use multi pack of credits (2,3,4) |

The response will be a JSON object (unless `html` or `redirect` are set to 1) containing the following properties:
- `tst_url`: the URL that will automatically start the specified test
- `pla_tst_id`: integer ID of this test for this candidate in Isograd's platform

Note: if the candidate has an unfinished test and `act_id` is set to `2`, the response will contains the unfinished test's details. 

**Errors**

| error_code  | error_message  |
|---|---|
| 106 | This `tst_frm_id` is not allowed |
| 107 | This candidate does not exist |
| 301 | You have no more credits for this type of test |
| 402 | The candidate has already taken `max_num_tst` for this `tst_frm_id` |

> 💡 Note: Most systems connecting to the Isograd platform will set the optional `redirect` parameter to true as it allows to have the standard expected behaviour: the candidate clicks on a link in the LMS and the test starts automatically.

### Create a candidate and take test

| Parameter  | Required  | Value |
|---|---|---|
| act_id | 🟩 | 8 |

The system will perform successively the Create Candidate and Add a Test actions described above. Consequently, the return structure will be similar to Add a Test action and all the parameters required or optional in the Add Test and Create Candidate actions may be provided.

### Add/remove online proctoring to a test

| Parameter  | Required  | Value |
|---|---|---|
| act_id | 🟩 | 16: add online proctoring, 17: remove online proctoring |
| pla_tst_id | 🟩  | ID of the test that was returned when the test was created |

The response will be a JSON object containing no specific property.

**Errors**

| error_code  | error_message  |
|---|---|
| 303 | You have no proctoring credit left |


### Delete a test

| Parameter  | Required  | Value |
|---|---|---|
| act_id | 🟩 | 23 |
| pla_tst_id | 🟩  | ID of the test that was returned when the test was created |

The response will be a JSON object containing no specific property.

**Errors**

| error_code  | error_message  |
|---|---|
| 304 | The test is already started or does not exist |


### Add a confirmation test to a completed test

| Parameter  | Required  | Value |
|---|---|---|
| act_id | 🟩 | 58 |
| pla_tst_id | 🟩  | ID of the test for which a confirmation test shoul be created (the test has to be already completed) |
| redirect | 🟠 | If set to 1: The response will be a HTTP 302 status redirecting to the URL of the test start page |

The response will be a JSON object (unless `redirect` is set to 1) containing the following properties:
- `tst_url`: the URL to start the confirmation test.

  
### Update a candidate

| Parameter  | Required  | Value |
|---|---|---|
| act_id | 🟩 | 51 |
| can_id | 🟩  | ID of the candidate that was returned when the candidate was created |
| gen_id | 🟠  | 1: male, 2: female, 3: not provided |
| ema | 🟠 | Candidate's email address |
| fst_nam | 🟠 | Candidate's first name |
| lst_nam | 🟠 | Candidate's last name |
| lan_id | 🟠 | Language code. See the [appendix](#language-codes) |

### Delete a candidate

| Parameter  | Required  | Value |
|---|---|---|
| act_id | 🟩 | 24 |
| can_id |  🟠  | ID of the candidate to be deleted, must be provided except if ema is provided |
| ema | 🟠 | ema of the candidate to be deleted, must be provided except if can_id is provided |

The response will be a JSON object containing only a success property.

## Emails
### Send registration email

This action enables to send a registration email to the candidate immediately or in the future. A specific email sender may be provided.


| Parameter  | Required  | Value |
|---|---|---|
| act_id | 🟩 | 42 |
| pla_tst_ids | 🟠  | The pla_tst_ids of the tests for which the registration email will be sent. Please note that if you dont fill this field and the candidate is registered for multiple assessments, the email content will include all the pending tests to which the candidate is registered |
| can_id |  🟠 | The can_id returned when the candidate was created |
| mai_tem_id | 🟩 | The id of the mail template that you want to use (can be found in the URL while editing the template on the plateform)|
| sen_dat | 🟠 | The date on which the email should be send, format should be  YYYY-MM-DD|
| for_sen | 🟠 | If the value is 1 and a sen_dat is provided and if it is today or in the past then send the email immediately|
| ema_sen | 🟠 | An approved email sender with a verified status|

A can_id or a pla_tst_id must be provided, if both are provided, the pla_tst_id will be used.

The response JSON only includes a success field

## Results
### Get PDF results

| Parameter  | Required  | Value |
|---|---|---|
| act_id | 🟩 | 6 |
| tst_frm_id | 🟩  | The test identifier. For compatibility reasons this parameter may be named rea_tst_id until 31/12/2025. | |
| ema | 🔷 | Candidate's email address |
| pla_tst_id | 🟠 | The ID that was returned when the test was created. If this parameter is not provided, the system will look for the last test taken. |
| lan_id | 🟠 | Language code used for the report. See the [appendix](#language-codes) |
| redirect | 🟠 | 1: if you wish to receive a HTTP 302 status redirecting you to the URL of the PDF report |

The response will be a JSON object (unless `redirect` is set to 1) containing the following properties:
- `pdf_url`: the URL of the PDF report

**Errors**

| error_code  | error_message  |
|---|---|
| 106 | This value for `tst_frm_id`  is not authorized |
| 107 | Candidate does not exist |
| 501 | Candidate has not taken this test |
| 502 | Test is not finished |
| 601 | Test must be an assessment |

### Get results as a JSON

| Parameter  | Required  | Value |
|---|---|---|
| act_id | 🟩 | 4 |
| tst_frm_id | 🟩  | The test identifier. For compatibility reasons this parameter may be named rea_tst_id until 31/12/2025. | |
| ema | 🔷 | Candidate's email address |
| pla_tst_id | 🟠 | The ID that was returned when the test was created. If this parameter is not provided, the system will look for the last test taken. |

The response will be a JSON object containing the following properties:

- `gra_typ_id`: an integer representing the grade type (see the possible values below)
- `num_val`:  a number representing the numeric value of the grade
- `max_val`: a number representing the maximum value corresponding to the `gra_typ_id` 
- `txt_des`: a text description of the grade (e.g. "550/1000" or "Intermediate – 3/5")
- `no_sto_tim_spe`: the difference expressed in seconds between the start time and the end time of the test. (This time may not correspond to the time spent by the candidate on the test as he or she may have the option in some cases to stop the test and restart it later).

The potential values for `gra_typ_id` are:

| Value  |  Description |
|---|---|
| 1 | Level on a 1 to 5 scale |
| 2 | Score on 1000 computed through Item Response Theory | 
| 3 | Score on 1000 computed by averaging scores on each question |
| 4 | Number of correct answers |
| 5 | Range of score 1-350, 350-700, 700-1000 computed using IRT |
| 15 | Score on 100 computed by averaging scores on each question |

### Get results details as a JSON

| Parameter  | Required  | Value |
|---|---|---|
| act_id | 🟩 | 5 |
| tst_frm_id | 🟩  | The test identifier. For compatibility reasons this parameter may be named rea_tst_id until 31/12/2025. | |
| ema | 🔷 | Candidate's email address |
| pla_tst_id | 🟠 | The ID that was returned when the test was created. If this parameter is not provided, the system will look for the last test taken. |
| des_lan_id | 🟠 | Language code used for the details. See the [appendix](#language-codes) |

The response will be a JSON object containing the following properties:
- `des_lan_id`: the language used the result details. If no description is available in the requested language, English will be used by default.
- `result_details`: an array where the keys are the skill IDs and values are an array composed of values whose keys are:
    - `des`: skill description
    - (for tests based on IRT) `level`: Decimal number representing the level obtained by the candidate. A value of -1 means the candidate has not been tested on this skill.
    - (for tests based on IRT) `level_des`: Brief text description of the candidate's skills at this level.
    - (for tests based on IRT) `scalemin`: Minimum value of level
    - (for tests based on IRT) `scalemax`: Maximum value of level
    - (for tests NOT based on IRT) `successrate`: Success rate on questions belonging this skill. This is a decimal value equal to -1 or between 0 and 1. A value of -1 means the candidate has not been tested on this skill.

### Get test status

| Parameter  | Required  | Value |
|---|---|---|
| act_id | 🟩 | 10 |
| pla_tst_id | 🟩  | The ID that was returned when the test was created 

The response will be a JSON object containing the following properties:
- `sta_id`: an integer representing the status of the test:
    - 1: not started
    - 2: started
    - 3: completed
    - 4: marking pending (for tests that include manual marking)

**Errors**

| error_code  | error_message  |
|---|---|
| 501 | This `pla_tst_id` does not exist |

## Administration
### Create a session

| Parameter  | Required  | Value |
|---|---|---|
| act_id | 🟩 | 11 |
| des | 🟩  | Description of the session |
| sta_dat | 🟩  | Starting date of the session (format YYYY-MM-DD hh:mm) |
| end_dat | 🟩 | End date of the session (format YYYY-MM-DD hh:mm) |
| psw | 🟠 | Password of the session |

The response will be a JSON object containing the following properties:
- `ses_id`: An integer representing the unique ID of the session

### Anonymize a candidate

| Parameter  | Required  | Value |
|---|---|---|
| act_id | 🟩 | 12 |
| ema | 🟩  | The email of the candidate |

Their first name, last name and email address will be replaced by numbers in Isograd's database. This also affects their diploma(s).

The response will be a JSON object containing no specific property.

### Log-in as an administrator

| Parameter  | Required  | Value |
|---|---|---|
| act_id | 🟩 | 13 |
| use_ema_for_adm | 🟠 | 1: if you want want to login as a specific administrator. If not set, the system will pick a random administrator |
| ema | 🟠 | The admin email if `use_ema_for_adm` is set to 1 |

> ⚠️ **Be very careful** not to provide access to this request to candidates as they would access your account on the platform and would be able to create candidates, see results, add tests, etc.
>
> ⚠️ This method, like all others, includes an optional `sub_com_id` parameter for customers who use sub accounts. If the parameter `sub_com_id` is not provided,  the sub account administrator will be logged as a master account administrator which could raise **serious data confidentiality issue**.

The response will be a HTTP redirection to the Admin module

### Log-in as a candidate

| Parameter  | Required  | Value |
|---|---|---|
| act_id | 🟩 | 38 |
| ema | 🔷 | Candidate email |


The response will be a HTTP redirection to the candidate's account page, where they can access their:
- Tests to take
- Diplomas
- Assessment reports
- Questions and correct answers for assessment tests.


### Get available tests list

| Parameter  | Required  | Value |
|---|---|---|
| act_id | 🟩 | 14 |
| lan_id | 🟠 | Code of the candidate's language. See the [appendix](#language-codes) |

The response will be a JSON object containing the following properties:
- `tests`: An array of the IDs of the allowed tests.

### Get available credits

| Parameter  | Required  | Value |
|---|---|---|
| act_id | 🟩 | 15 |

The response will be a JSON object containing the following properties:
- `credits`: an array of that contains for each type of credits the following properties:
    - `des`: credit type description
    - `amount`: credit amount

# Appendices
## Language codes
The following values must be used to specify languages:
- 1: French
- 2: English
- 3: German
- 4: Dutch
- 5: Spanish
- 6: Italian
- 7: Greek
- 8: Arabic

## Distributor accounts
Distributor accounts can perform actions on sub accounts that belong to them using the credentials of the distributor account. In such case an additional parameter `sub_com_id` can be added to the request to specify on which sub account the action should be performed.

This integer `sub_com_id` **can be added to all actions described in the section Action.**

## LTI - Grades via Basic Outcome Service
Isograd LTI provider service can return grades through the LTI 1.1 Basic Outcome Service. This allows your system to receive the grade of the candidate automatically at the end of the test. If you wish to receive the grade through Basic Outcome Service, please contact Isograd support to request its activation on your account.

LTI Basic Outcome Service only allows a grade as a decimal number between 0 and 1. The different grade
types are converted into a decimal number between 0 and 1 according to the below rules:
- LEVEL_GRADE_TYPE = 1, the level divided by 10
- SCORE_GRADE_TYPE = 2, the score divided by 1000
- AVERAGE_GRADE_TYPE = 3, the score divided by 1000
- NUM_CORRECT_GRADE_TYPE = 4, the percentage of correct answers that is: number of correct
answers / number of questions
- RANGE_GRADE_TYPE = 5, the middle value of the relevant range divided by 1000
- ON_100_AVERAGE_GRADE_TYPE = 15, the score divided by 1000
  </div>
