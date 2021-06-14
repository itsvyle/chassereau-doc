# DB Documentation

## Structure
**The request data:**
An englobing node (named `commands`):
```xml
<commands>
  <!-- One or more commands here -->
</commands>
```

**The result:**
```xml
<results status="1" datestarted="210405012300" datecompleted="210405012300">
  <!-- Results of individual commands -->
</results>
```
**Note**: The `status` attribute of the `<results>` will be '1' ONLY if there were no mistakes while the execution

**Each command**: Each command will have at least one attribute, `status`, which indicates if the command was executed with success
```xml
<commandname status="0|1" ...other attributes/>
```

### Extra information on status
The status attribute of each command clearly indicates if the request was entirely fufilled, without any errors, and that the request's results are saved and visible in the databases.
The status will **not** be 1 if:
* The request could not be made due to authorization restrictions
* The SQL command encountered any error, for example if the databases are full
* The fields are invalid
* The required XML syntax of the command is not present

**Note that the status of the `<result>` will be 1 only if all the commands have `status="1"`**

## Authentication
**Note**: Some commands might not require the caller to be authenticated

### To login
Add this command inside of `<commands>`:
```xml
<login user_id="your_user_id" password="your_password" session_duration_minutes="120"/>
```

If the credentials are correct, it should return something like this:
```xml
<login status="1" loggedin="1" session_token="a_new_session_token" session_end="the_sessions_expiration_date" session_end_in="time_to_expiration_in_seconds" last_session_token="the_previous_session_token" last_session_start="the_previous_session_token_startdate">
   <approle>Role1</approle>
   <approle>Role2</approle>
</login>
```
Note that to use the newly created credentials for the other commands of the request, you need to add the `use="1"` attribute to the `<login>`. You will still need to precise the `app_role` attribute for the `<commands>` tag.

#### Using old session_token
By specifying the `use_last="1"` attribute in the `<login>` request, the caller can ask for the latest session token to be used, if it is not expired. This means that instead of systematically creating a new token, the server will do the following things :
* Check if the last session_token is still valid
* If YES: will return the last token, with all the data as usual
* If NOT: Will create a new token, and do everything as usual

### To execute requests
After that, for each query to the database, add the `session_token` and `app_role` attributes to the englobing node, as so :
```xml
<commands session_token="a_new_session_token" app_role="Role2">
  <!-- Your commands here -->
</commands>
```

## Commands
For now, these are the possible commands:
* insert
* insertupdate
* update
* [select](#select)
* delete

## Global options
These are options that will be used in more than one type of command

### Keys:
```xml
<commands>
  <insert table="xxx">
    <keys NO="autoincrement" YYY="constantvalue"/>
    <fields zzz="123"/>
  </insert>
</commands>
```

_Possible key values_:
* autoincrement : Used in an `<insert>` command, for the autoincrement columns

The keys will be returned at the end. Example:
```xml
<results status="1">
  <insert table="xxx" NO="1" YYY="constantvalue"></insert>
</results>
```

### Fields:
```xml
<commands>
  <insert table="xxx">
    <fields zzz="123"/>
  </insert>
</commands>
```
For each command
* insert : The column values of the row that is added
* update : The new values
* select : The fields to be fetched. Then, the attribute value is the type of the field (xml, string, long)

**Note**: `<xmlfield XML_FIELD="string"><!-- XML HERE --></xmlfield>` can also be used to insert xml

### systemfields
```xml
<commands>
  <insert table="xxx">
    <fields zzz="123"/>
    <systemfields date="systemdate"/>
  </insert>
</commands>
```
These field's values are automatically set by DB.php
_Possible values_:
* systemdate : the current system date
* mastersystemdate : the date at which the request started
* lastinsertid : The last sql insert id

### Where:
```xml
<commands>
  <select table="xxx">
    <fields zzz="string"/>
    <where>
      <condition table="xxx" field="zzz" sign="equal" value="123" type="string"/>
    </where>
  </select>
</commands>
```
A SQL Where is generated from the `<where>` structure.

## Select
### Extra options :

#### orderby:
```xml
<orderby field1="NO" field2="NAME"/>
```
orderby is used to precise by which column the rows are sorted.

**Note**: `ASC` can be added after the field name to reverse sort 

#### groupby:
```xml
<groupby field1="NO" field2="NAME"/>
```
groupby is used to precise by which column the rows are grouped (executes the sql `GROUPBY` option).

#### pagination
```xml
<pagination page="0" size="50"/>
```

#### join:
```xml
<join type="LEFT" table="QA_RESPONSES" condition="QA_RESPONSES.QUESTION_ROOTNO=QA_QUESTIONS.ROOTNO" />
```
Executes the sql `JOIN ON <type>`
