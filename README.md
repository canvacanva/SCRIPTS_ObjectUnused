# SCRIPTS_ObjectUnused

## COMPUTERS
```
## BASIC PARAMETERS
$TYPE = "PC"
$SearchBase="OU=XX,OU=Computer Accounts,DC=XX,DC=DOMAAIN,DC=it"
$EXPIRED = 30
$smtpServer = "relay"
$from = "$TYPE"+"_unused@domain"
$recipient = "unused_objects@domain"
$bodytemp = ""
$bodyfull = ""
 
#FIRST CHECK
Import-Module ActiveDirectory
$CLIENTS = Get-ADComputer -Filter * -SearchBase $SearchBase -Properties * | Sort whenchanged | Select Name, whenchanged, lastlogon , description
$countprocessed=${CLIENTS}.Count
$TODAY = (get-date)
$ALERTDAYS = $TODAY.AddDays(-$EXPIRED)
$textEncoding = [System.Text.Encoding]::UTF8
$subject="UNUSED $TYPE for more than $EXPIRED Days"

#RUN APPLICATION
foreach ($CLIENTS in $CLIENTS) {
    $NAME = $CLIENTS.Name
    $CHANGED = $CLIENTS.whenchanged
    $CHANGEDAYS = ($TODAY - $CHANGED).Days
    $DESCRIPT = $CLIENTS.description
    #echo $NAME
    #echo $CHANGED
    #echo $TODAY
    #echo $ALERTDAYS
    #echo $CHANGEDAYS
    echo $DESCRIPT

    if ($CHANGEDAYS -gt $EXPIRED) {

        # Email Body Set Here, Note You can use HTML, including Images.
        $bodytemp="
        <p>Verify $TYPE $NAME $DESCRIPT. It has no connection from $CHANGEDAYS days; Last connection on $CHANGED.</p>
        "
        $bodyfull = $bodyfull + $bodytemp
        #echo $bodyfull
        echo "Send alert email for "$NAME
        }
        else
        {
        echo $NAME" OK - last log on "$CHANGED
        }
    }
if ($bodyfull -eq ""){
$subject="$TYPE last change value is OK"
$bodyfull = "No $TYPE to verify. Unused days alert = $EXPIRED days."
Send-Mailmessage -smtpServer $smtpServer -from $from -to $recipient -subject $subject -body $bodyfull -bodyasHTML -priority High -Encoding $textEncoding -ErrorAction Stop -ErrorVariable err
}
else{
Send-Mailmessage -smtpServer $smtpServer -from $from -to $recipient -subject $subject -body $bodyfull -bodyasHTML -priority High -Encoding $textEncoding -ErrorAction Stop -ErrorVariable err
}
```

## USERS
```
## BASIC PARAMETERS
$TYPE = "USERS"
$SearchBase="OU=XX,DC=corp,DC=DOMAIN,DC=it"
$EXPIRED = 25
$smtpServer = "relay"
$from = "$TYPE"+"_unused@domain"
$recipient = "unused_objects@domain"
$bodytemp = ""
$bodyfull = ""
$GROUPNOCLIENT = get-adgroup GRP_NO-CLIENT_USERS
 
#FIRST CHECK - OFFICE
Import-Module ActiveDirectory
$USERS = Get-ADUser -Filter {(enabled -eq $true)} -SearchBase $SearchBase -Properties * |Where-Object {$GROUPNOCLIENT.DistinguishedName -notin $_.memberof -and $_.DistinguishedName -notlike "*OU=excluded1*" -and $_.DistinguishedName -notlike  "*OU=excluded2*"} | Sort whenchanged | Select Name, whenchanged, lastlogon , description, sAMAccountName
$countprocessed=${USERS}.Count
$TODAY = (get-date)
$ALERTDAYS = $TODAY.AddDays(-$EXPIRED)
$textEncoding = [System.Text.Encoding]::UTF8
$subject="UNUSED $TYPE for more than $EXPIRED Days"

#RUN APPLICATION OFFICE
foreach ($USERS in $USERS) {
    $NAME = $USERS.Name
    $CHANGED = $USERS.whenchanged
    $SAMAC = $USERS.sAMAccountName
    $CHANGEDAYS = ($TODAY - $CHANGED).Days
    $DESCRIPT = $USERS.description
    #echo $NAME
    #echo $CHANGED
    #echo $TODAY
    #echo $ALERTDAYS
    #echo $CHANGEDAYS
    #echo $DESCRIPT

    if ($CHANGEDAYS -gt $EXPIRED) {

        # Email Body Set Here, Note You can use HTML, including Images.
        $bodytemp="
        <p>Verify $TYPE $NAME ($SAMAC) $DESCRIPT. It has no connection from $CHANGEDAYS days; Last connection on $CHANGED.</p>"
        $bodyfull = $bodyfull + $bodytemp
        #echo $bodyfull
        echo "Send alert email for "$NAME
        }
        else
        {
        echo $NAME" OK - last log on "$CHANGED
        }
    }


if ($bodyfull -eq ""){
$subject="$TYPE last change value is OK"
$bodyfull = "No $TYPE to verify. Unused days alert = $EXPIRED days."
Send-Mailmessage -smtpServer $smtpServer -from $from -to $recipient -subject $subject -body $bodyfull -bodyasHTML -priority High -Encoding $textEncoding -ErrorAction Stop -ErrorVariable err
}
else{
Send-Mailmessage -smtpServer $smtpServer -from $from -to $recipient -subject $subject -body $bodyfull -bodyasHTML -priority High -Encoding $textEncoding -ErrorAction Stop -ErrorVariable err
}
```
