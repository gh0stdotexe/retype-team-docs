Determing how your validators delegators have voted can be accomplished by running a one-liner command, running an export command that will create a CSV file of the voter data, and then importing that data into Google Sheets and `counting if` 's:

Walk through pages of delegations to a validator (enter `valoper` address):

```shell
# grab delegator voting data for a given validator (use valoper)

for i in $(seq 1 11); do junod q staking delegations-to junovaloper1xwazl8ftks4gn00y5x3c47auquc62ssuvynw64 --limit 100 --page $i --output json | jq -r .delegation_responses[].delegation.delegator_address >> delegators.txt; sleep 5; done
```

Create CSV file from data grabbed:

```shell
# creating a CSV from data grabbed from above command

for i in $(cat delegators.txt); do echo "$i, $(junod q gov vote 18 $i --output json 2> /dev/null | jq .option 2> /dev/null)"; done > out.csv
```

Import into Google Sheets and "Count `if` 's "

Example:

<https://docs.google.com/spreadsheets/d/1ARjv4bsbT5HRZLfhqhMl8oQMp8tP3D5m-7HVo0X2a9M/edit?usp=sharing>
