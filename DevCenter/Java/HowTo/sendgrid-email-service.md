<properties linkid="dev-java-how-to-sendgrid-email-service" urldisplayname="SendGrid Email Service" headerexpose="" pagetitle="SendGrid Email Service - How To - Java - Develop" metakeywords="" footerexpose="" metadescription="" umbraconavihide="0" disquscomments="1"></properties>

# How to Send Email Using SendGrid from Java

This guide demonstrates how to perform common programming tasks with the
SendGrid email service on Windows Azure. The samples are written in
Java. The scenarios covered include **constructing email**, **sending
email**, **adding attachments**, **using filters**, and **updating
properties**. For more information on SendGrid and sending email, see
the [Next steps][] section.

## Table of Contents

-   [What is the SendGrid Email Service?][]
-   [Create a SendGrid account][]
-   [How to: Use the javax.mail libraries][]
-   [How to: Create an email][]
-   [How to: Send an email][]
-   [How to: Add an attachment][]
-   [How to: Use filters to enable footers, tracking, and analytics][]
-   [How to: Update email properties][]
-   [How to: Use additional SendGrid services][]
-   [Next steps][]

## <a name="bkmk_WhatIsSendGrid"> </a>What is the SendGrid Email Service?

SendGrid is a cloud-based email service that provides reliable email
delivery, scalability, and real-time analytics along with flexible APIs
that make custom integration easy. Common SendGrid usage scenarios
include:

-   Automatically sending receipts to customers.
-   Administering distribution lists for sending customers monthly
    e-fliers and special offers.
-   Collecting real-time metrics for things like blocked e-mail, and
    customer responsiveness.
-   Generating reports to help identify trends.
-   Forwarding customer inquiries.

For more information, see [http://sendgrid.com][].

## <a name="bkmk_CreateSendGridAcct"> </a>Create a SendGrid account

To get started with SendGrid, evaluate pricing and sign up here:
[http://sendgrid.com/pricing.html][]. Windows Azure customers receive a
special offer of 25,000 free emails per month from SendGrid. To sign-up
for this offer, or get more information, please visit
[http://www.sendgrid.com/azure.html][]. For more detailed information on
the signup process, see the SendGrid documentation at
[http://docs.sendgrid.com/documentation/get-started][]. For information
about additional services provided by SendGrid, see
[http://sendgrid.com/features][].

## <a name="bkmk_HowToUseJavax"> </a>How to: Use the javax.mail libraries

Obtain the javax.mail libraries, for example from
[http://www.oracle.com/technetwork/java/javamail][] and import them into
your code. At a high-level, the process for using the javax.mail library
to send email using SMTP is to do the following:

1.  Specify the SMTP values, including the SMTP server, which for
    SendGrid is smtp.sendgrid.net.

        public class MyEmailer {
	       private static final String SMTP_HOST_NAME = "smtp.sendgrid.net";
	       private static final String SMTP_AUTH_USER = "your_sendgrid_username";
           private static final String SMTP_AUTH_PWD = "your_sendgrid_password";
        
		   public static void main(String[] args) throws Exception{
         	  new MyEmailer().SendMail();
           }
        
		   public void SendMail() throws Exception
           {
              Properties properties = new Properties();
           	  properties.put("mail.transport.protocol", "smtp");
           	  properties.put("mail.smtp.host", SMTP_HOST_NAME);
           	  properties.put("mail.smtp.port", 587);
           	  properties.put("mail.smtp.auth", "true");
           	  // …

2.  Extend the <span class="auto-style1">javax.mail.Authenticator</span>
    class, and in your implementation of the
    <span class="auto-style1">getPasswordAuthentication</span> method,
    return your SendGrid user name and password.  

        private class SMTPAuthenticator extends javax.mail.Authenticator {
        public PasswordAuthentication getPasswordAuthentication() {
           String username = SMTP_AUTH_USER;
           String password = SMTP_AUTH_PWD;
           return new PasswordAuthentication(username, password);
        }

3.  Create an authenticated email session through a
    <span class="auto-style1">javax.mail.Session</span> object.  

        Authenticator auth = new SMTPAuthenticator();
        Session mailSession = Session.getDefaultInstance(properties, auth);

4.  Create your message and assign **To**, **From**, **Subject** and
    content values. This is shown in the [How To: Create an Email](#bkmk_HowToCreateEmail) section.
5.  Send the message through a
    <span class="auto-style1">javax.mail.Transport</span> object. This
    is shown in the [How To: Send an Email][How to: Send an Email]
    section.

## <a name="bkmk_HowToCreateEmail"> </a>How to: Create an email

The following shows how to specify values for an email.

    MimeMessage message = new MimeMessage(mailSession);
    Multipart multipart = new MimeMultipart("alternative");
    BodyPart part1 = new MimeBodyPart();
    part1.setText("Hello, Your Contoso order has shipped. Thank you, John");
    BodyPart part2 = new MimeBodyPart();
    part2.setContent(
		"<p>Hello,</p>
		<p>Your Contoso order has <b>shipped</b>.</p>
		<p>Thank you,<br>John</br></p>",
		"text/html");
    multipart.addBodyPart(part1);
    multipart.addBodyPart(part2);
    message.setFrom(new InternetAddress("john@contoso.com"));
    message.addRecipient(Message.RecipientType.TO,
       new InternetAddress("someone@example.com"));
    message.setSubject("Your recent order");
    message.setContent(multipart);

## <a name="bkmk_HowToSendEmail"> </a>How to: Send an email

The following shows how to send an email.

    Transport transport = mailSession.getTransport();
    // Connect the transport object.
    transport.connect();
    // Send the message.
    transport.sendMessage(message, message.getRecipients(Message.RecipientType.TO));
    // Close the connection.
    transport.close();

## <a name="bkmk_HowToAddAttachment"> </a>How to: Add an attachment

The following code shows you how to add an attachment.

    // Local file name and path.
    String attachmentName = "myfile.zip";
    String attachmentPath = "c:\\myfiles\\"; 
    MimeBodyPart attachmentPart = new MimeBodyPart();
    // Specify the local file to attach.
    DataSource source = new FileDataSource(attachmentPath + attachmentName);
    attachmentPart.setDataHandler(new DataHandler(source));
    // This example uses the local file name as the attachment name.
    // They could be different if you prefer.
    attachmentPart.setFileName(attachmentName);
    multipart.addBodyPart(attachmentPart);

## <a name="bkmk_HowToUseFilters"> </a>How to: Use filters to enable footers, tracking, and analytics

SendGrid provides additional email functionality through the use of
*filters*. These are settings that can be added to an email message to
enable specific functionality such as enabling click tracking, Google
analytics, subscription tracking, and so on. For a full list of filters,
see [Filter Settings][].

-   The following shows how to insert a footer filter that results in
    HTML text appearing at the bottom of the email being sent.

        message.addHeader("X-SMTPAPI", 
			"{\"filters\": 
			{\"footer\": 
			{\"settings\": 
        	{\"enable\":1,\"text/html\": 
			\"<html><b>Thank you</b> for your business.</html>\"}}}}");

-   Another example of a filter is click tracking. Let’s say that your
    email text contains a hyperlink, such as the following, and you want
    to track the click rate:

        messagePart.setContent(
			"Hello,
			<p>This is the body of the message. Visit 
			<a href='http://www.contoso.com'>http://www.contoso.com</a>.</p>
			Thank you.", 
        	"text/html");

-   To enable the click tracking, use the following code:

        message.addHeader("X-SMTPAPI", 
			"{\"filters\": 
			{\"clicktrack\": 
			{\"settings\": 
        	{\"enable\":1}}}}");

## <a name="bkmk_HowToUpdateEmail"> </a>How to: Update email properties

Some email properties can be overwritten using **set*Property*** or
appended using **add*Property***.

For example, to specify **ReplyTo** addresses, use the following:

    InternetAddress addresses[] = 
		{ new InternetAddress("john@contoso.com"),
          new InternetAddress("wendy@contoso.com") };
    
	message.setReplyTo(addresses);

To add a **Cc** recipient, use the following:

    message.addRecipient(Message.RecipientType.CC, new 
    InternetAddress("john@contoso.com"));

## <a name="bkmk_HowToUseAdditionalSvcs"> </a>How to: Use additional SendGrid services

SendGrid offers web-based APIs that you can use to leverage additional
SendGrid functionality from your Windows Azure application. For full
details, see the [SendGrid API documentation][].

## <a name="bkmk_NextSteps"> </a>Next steps

Now that you’ve learned the basics of the SendGrid Email service, follow
these links to learn more.

* SendGrid in a Windows Azure Deployment: [How to Send Email Using SendGrid from Java in a Windows Azure Deployment](./sendgrid-email-service-on-azure.md)
* SendGrid Java information: <http://docs.sendgrid.com/documentation/get-started/integrate/examples/java-email-example-using-smtp/>
* SendGrid API documentation: <http://docs.sendgrid.com/documentation/api/>
* SendGrid special offer for Windows Azure customers: <http://sendgrid.com/azure.html>

  [Next Steps]: #bkmk_NextSteps
  [What is the SendGrid Email Service?]: #bkmk_WhatIsSendGrid
  [Create a SendGrid Account]: #bkmk_CreateSendGridAcct
  [How to: Use the javax.mail libraries]: #bkmk_HowToUseJavax
  [How to: Create an Email]: #bkmk_HowToCreateEmail
  [How to: Send an Email]: #bkmk_HowToSendEmail
  [How to: Add an Attachment]: #bkmk_HowToAddAttachment
  [How to: Use Filters to Enable Footers, Tracking, and Analytics]: #bkmk_HowToUseFilters
  [How to: Update Email Properties]: #bkmk_HowToUpdateEmail
  [How to: Use Additional SendGrid Services]: #bkmk_HowToUseAdditionalSvcs
  [http://sendgrid.com]: http://sendgrid.com
  [http://sendgrid.com/pricing.html]: http://sendgrid.com/pricing.html
  [http://www.sendgrid.com/azure.html]: http://www.sendgrid.com/azure.html
  [http://docs.sendgrid.com/documentation/get-started]: http://docs.sendgrid.com/documentation/get-started
  [http://sendgrid.com/features]: http://sendgrid.com/features
  [http://www.oracle.com/technetwork/java/javamail]: http://www.oracle.com/technetwork/java/javamail/index.html
  [Filter Settings]: http://docs.sendgrid.com/documentation/api/smtp-api/filter-settings/
  [SendGrid API documentation]: http://docs.sendgrid.com/documentation/api/
  [http://docs.sendgrid.com/documentation/get-started/integrate/examples/java-email-example-using-smtp/]:
    http://docs.sendgrid.com/documentation/get-started/integrate/examples/java-email-example-using-smtp/
  [http://sendgrid.com/azure.html]: http://sendgrid.com/azure.html
