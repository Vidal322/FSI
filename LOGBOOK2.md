# Trabalho realizado nas Semanas #2 e #3

## Identificação

- CVE-2017-5638
- CVE-2021-34646
- item3
- item4

## Catalogação

- (CVE-2017-5638):
The Jakarta Multipart parser in Apache Struts 2 2.3.x before 2.3.32 and 2.5.x before 2.5.10.1 has incorrect exception handling and error-message generation during file-upload attempts, which allows remote attackers to execute arbitrary commands via a crafted Content-Type, Content-Disposition, or Content-Length HTTP header, as exploited in the wild in March 2017 with a Content-Type header containing a #cmd= string.
10.0 Critical

- (CVE-2021-34646):
Versions up to, and including, 5.4.3, of the Booster for WooCommerce WordPress plugin are vulnerable to authentication bypass via the process_email_verification function due to a random token generation weakness in the reset_and_mail_activation_link function found in the ~/includes/class-wcj-emails-verification.php file. This allows attackers to impersonate users and trigger an email address verification for arbitrary accounts, including administrative accounts, and automatically be logged in as that user, including any site administrators. This requires the Email Verification module to be active in the plugin and the Login User After Successful Verification setting to be enabled, which it is by default.
9.8 Critical

- item3
- item4

## Exploit

- (CVE-2017-5638):
   https://www.exploit-db.com/exploits/41570
   Affected App: Apache
   Created:	Mar 09, 2017
   Affected Products:
   - Struts 2.3.5 - Struts 2.3.31
   - Struts 2.5 - Struts 2.5.10
   System Compromise: Remote attackers can gain control of vulnerable systems.

   (possible workaround): 
   Implement a Servlet filter which will validate Content-Type and throw away request with suspicious 
   values not matching multipart/form-data.
   Other option is to remove the File Upload Interceptor from the stack, just define your own custom stack and set it as a default

- (CVE-2021-34646):
  Exploit Title: WordPress Plugin WooCommerce Booster Plugin 5.4.3 - Authentication Bypass
  Date: 2021-09-16
  Exploit Author: Sebastian Kriesten
  Affected Plugin: Booster for WooCommerce
  Plugin Slug: woocommerce-jetpack
  Affected Versions: <= 5.4.3
  Exploit(https://www.exploit-db.com/exploits/50299):
  1:
  Goto: https://target.com/wp-json/wp/v2/users/
  Pick a user-ID (e.g. 1 - usualy is the admin)

  2:
  Attack with: ./exploit_CVE-2021-34646.py https://target.com/ 1

  3:
  Check-Out  out which of the generated links allows you to access the system

- item3
- item4

## Ataques

- (CVE-2017-5638):
  Guest Self Registration - Cisco ISE portal;
  It is possible to perform a RCE attack with a malicious Content-Type value. If the Content-Type value isn't valid an exception
  is thrown which is then used to display an error message to a user.

- (CVE-2021-34646):
  Wordfence Premium users received a firewall rule to protect against any exploits targeting this vulnerability on July 30, 2021.
  Sites still using the free version of Wordfence will receive the same protection on August 29, 2021.

   Initially reached out to the plugin vendor on July 30, 2021. After receiving confirmation of an appropriate communication channel  a few days later on August 2, 2021, it was provided the full disclosure details. The vendor quickly acknowledged the report and a patch was released on August 11, 2021 in version 5.4.4.

- item3
- item4
