# enrollmentComplete
A script for setting up a Mac with Jamf Pro, to be executed initially by the Jamf Pro event/trigger enrollmentComplete

## Script: OnBoarded-Checklist

**OnBoarded-Checklist** is a script for monitoring the installation status of apps installed my methods where other options are not available (essentially Jamf Pro added Self Service Onboarding, however at release of the feature, it is unable to include Jamf App Installers from the Jamf App Catalog). This script will use the Developer Team ID to help it identify if an application has completed installation.  
This is a work in progress, as testing is ongoing, hopefully Jamf will add Jamf App Installers to Self Service OnBoarding quickly making this no longer required.

***

## Script: enrollmentComplete

#### Files:

**enrollmentComplete:** A sanitised, cleaned up, and public version of the script I use with Jamf Pro to manage enrolments.  
**nz.co.wak.enrollmentComplete.json:** A json file for creating a config profile in Jamf Pro for the enrollmentComplete script.  
**Note:** For non-americans (like myself), I have used the American spelling of enrollment for consistency with Jamf Pro (as an American product).

#### Requirements:

**Jamf Pro 10.49 minimum(?):** Essentially the first version of Jamf Pro to support the API client id/secrets intended for supporting API access instead of user accounts.  
An **Active Directory** that supports legacy LAPS (through macOSLAPS, and unless configured otherwise).  
**macOS 12 Monterey:** not tested on anything earlier, but realistically the minimums for **Installomator**, **mkuser**, **swiftDialog**, and **macOSLAPS** all apply.

The enrollmentComplete script is designed to be the first and only script executed by the only policy with the Enrollment trigger.  
For a Manual Enrollment (or re-enrollment), a Secure Token enabled account is required, preferably one that is also admin, however it will promote to admin if necessary.

#### Script Setup:

**$4:** Must contain the domain name used for the config profile (defaults to nz.co.wak).  
**$5:** Contains the temporary admin password (defaults to setup).  
**$6:** Contains the LAPS admin account initial (first) password (defaults to the temporary admin password).  
**$7 & $8:** Must contain the API Client ID, and API Client Secret - These are used to lookup the Jamf managed admin account, to help make sure all admin accounts get Secure Tokens, and in the case of a re-enrollment, resets the Jamf managed admin account password to match to make sure Jamf is able to work with the account as required.
**$9:** Contains the password used for authenticated email sending.
**$5-9:** must all be base64 encoded. from the commandline do: echo "password or secret here" | base64,

#### Config. Profile Setup:

**managementID:** must be set to $MANAGEMENTID, this is required to look up the Jamf managed admin account details.  
**jssID:** must be set to $JSSID, this is used in email to link back to the computer record.

**tempAdmin:** Used to help with managing accounts and password, and for PreStage Enrollments, also setup to log into automatically to enable completion the computer setup. Defaults to setup_admin.  
**tempName:** real name for setup_admin, defaults to Setup Admin.  
**lapsAdmin:** this should be the default admin account used by IT staff, that they can look up using an appropriate LAPS tool. Defaults to laps_admin.  
**lapsName:** real name for laps_admin, defaults to LAPS Admin.

**adminPicture:** image file used by mkuser to add to admin accounts.  
**dialogIcon:** image file used in dialogs when communicating with users, such as the login box used to get the login of a Secure Token enabled user.

**requireNetwork:** wired, wireless, of off (default), some environments may have a require for the kind of network being used, this will cause the script to pause or provide warnings based on the kind of network connection in use.

**corpName:** display name identifying the organisation to which the computer belongs, used in network notices. Defaults to 'The Service Desk'.  
**serviceName:** display name identifying the organisations department providing support, used in emails. Defaults to 'Service Management'.

**systemTimeZone:** the time zone to set the computer to, for options see systemsetup -listtimezones. Defaults to the computers existing Time Zone setting.  
**systemTimeServer:** the time server to synchronise the time to. Defaults to the computers existing time server (usually time.apple.com).

#### Policies:

If you do not wish to use a policy below, set them to something that is not configured in your Jamf Pro instance, this will mean nothing will happen (if you choose to use the defaults for this, make sure the default names are not in use).  
**policyInstallomator:** policy used to install Installomator, I recommend using the script available from Installomator designed for this usage. Installomator is used for installing mkuser and swiftDialog, which are tools used elsewhere, but also having Installomator on the computer allows easier used with Self Service. Defaults to installInstallomator.  
**policyInitialFiles:** this policy should be used to install initial files such as the images used by swiftdialog and the admin accounts above. Defaults to installInitialFiles.  
**policyADUnbind:** this policy is used to unbind from Active Directory (or other login environment). Defaults to adUnbind.  
**policyComputerName:** this policy should be used to set the computer name before we bind to Active Directory. Defaults to fixComputerName.  
**policyADBind:** this policy is used to bind to Active Directory (or other login environment). Defaults to adBind.  
**policyCycleManagement:** this policy should be a policy within Jamf Pro set to rotate the management account password. Defaults to cyclePassword.  
**manualContinue:** when perform a manual enrollment, shall the script automatically continue to the next stage policy, ask the user, or just skip it. Default is to ask the user.  

**finalTextOne & finalTextTwo:** use these for setting how the text on the login window should be left when completed (lines one and two, using the com.apple.loginwindow LoginwindowText defaults setting).

#### Emails:

**emailFrom:** used as the email from address, if not set no email is sent.  
**emailTo:** this is the primary email address to send to, and will receive all emails.  
**emailErrors:** this address will only receive copies of error emails.  
**emailBCC:** this email address will receive all emails, however the address will be hidden the emailBCCFiller setting.  
**emailBCCFiller:** this can be anything, used to hide the actual to address when sending to the emailBCC address.  
**emailSMTP:** the email server & port <server.name:port> can be provided for sending authenticated email, leave unset for unauthenticated email sending.  
**emailAUTH:** is the authentication email address required for sending authenticated email, and will default to the emailFrom address.  
**emailFILE:** is the full path to a text file to include in the email, entering log here will include the log file. Default is not to include a file.
**emailSuccess:** true/false, decides if the completion email is to be sent.  
**emailJamfLog:** true/false, decides if we force an error back to Jamf Pro, so that the policy running this script fails and triggers the log to be emailed to anyone with emails for policy errors set. This setting is unaffected by whether the emailFrom address is set or not.

#### Exit signals (mostly error messages):

**0:** Successful execution, quiet logging. Jamf Pro doesn't send a policy error email, and records the policy as completed successfully on that computer.  
**All of the following will cause Jamf Pro to send a policy error email, and records the policy as failed on that computer.**  
**1:** Successful execution, noisy logging. Successful despite the error # being used to trigger the policy error email.  
**From here, all remaining signals are errors:**  
**2:** Defaults file not found after 5 minutes. This means the config. profile hasn't been configured, or the prefs domain is incorrectly set in $4.  
**3:** Jamf computer ID, and/or Jamf management ID from Jamf Pro are missing.  
**4:** API details missing.  
**5:** $TEMP_ADMIN failed to get a Secure Token during a PreStage Enrollment.  
**6:** $LAPS_ADMIN failed to get a Secure Token during a PreStage Enrollment.  
**7:** $JAMF_ADMIN failed to get a Secure Token during a PreStage Enrollment.  
**5,6,7** are checked in that order, and only for a PreStage Enrolment. None of these should happen, if any do, something is seriously broken.  
**8:** Unable to verify login details for $TEMP_ADMIN.  
**9:** Unable to verify login details for $JAMF_ADMIN.  
**10:** Unable to verify login details for $LAPS_ADMIN.
**8,9,10** are checked in that order before escrowing the BootStrap Token to verify both accounts have been setup correctly. Neither of these should happen, if either do, something is seriously broken.  
**11:** Unable to Get Jamf Managed Account username. Most likely one of the following are invalid: API details in $7 & $8 are incorrect, incorrect permissions on the API account, or the $MANAGEMENTID has not been set correctly.  
**12:** Unable to Get Jamf Managed Account password. Most likely just incorrect permissions on the API account, although realistically, this should not happen, as the account username should have failed first, so could imply something is seriously broken.  
**11,12** are checked while looking up the $JAMF_ADMIN details.  
**21:** Error occurred when attempting to set Time Zone & Syncronisation settings.  
**22:** Error occurred when attempting to install Rosetta (on Apple Silicon Macs only).  
**23:** Error occurred when attempting to install Installomator using a policy in Jamf Pro.  
**24:** Error occurred when attempting to install mkuser with Installomator.  
**25:** Error occurred when attempting to unbind from Active Directory using a policy in Jamf Pro.  
**26:** Error occurred when attempting to install swiftDialog with Installomator.  
**27:** Error occurred when attempting to install your initial files using a policy in Jamf Pro.  
**31:** Error occurred when attempting to perform the first jamf recon.  
**32:** Error occurred when attempting to reset the computer name using a policy in Jamf Pro.  
**33:** Error occurred when attempting to install macOSLAPS with Installomator.  
**34:** Error occurred when attempting to bind to Active Directory using a policy in Jamf Pro.  
**35:** Error occurred when attempting to perform the second jamf recon.  
**36:** Error occurred when attempting to cycle the $JAMF_ADMIN password using a policy in Jamf Pro.  
**37:** Error occurred when attempting to cycle the $LAPS_ADMIN password using macOSLAPS.  
**38:** Error occurred when attempting to perform the third jamf recon.  
The recon's are used to help change Jamf Pro change anything related to the policies executed between them based on what is happening (Computer name change, Binding, and password cycling for $LAPS_ADMIN and $JAMF_ADMIN).
