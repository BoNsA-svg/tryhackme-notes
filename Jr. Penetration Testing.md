
# Penetesting Fundamentals

## Penetration Testing Methodologies

Penetration tests can have a wide variety of objectives and target within scope. **The General theme** which are 5 :

| Stage                | Description                                                                                                                                                                                                      |
| -------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Info Gathering       | We collect info about the target from publically available resource, we use OSINT, and research, But keep in mind we don't yet ==scan== .                                                                        |
| Enumeration/Scanning | Discovering applications and services running on the systems, it could be finding a web server that might be potentially vulnerable                                                                              |
| Exploitation         | Once we find vulnerabilities on systems we try to leverage that to get inside.                                                                                                                                   |
| Privilege Escalation | Once exploited which is called a ==foothold==. We could go horizontally (to user's we the same permission) or go vertically (To administrator).                                                                  |
| Post Exploitation    | We could pivot to other hosts within the same network, we could collect information from the host that we are in, we have to cover our track, or report. **This part is mostly influenced by the ROE document**. |

The Table above is more of a general theme type of thing, but we got methodologies for every system lets begin :

#### OSSTMM
This framework is mainly used for systems, software, applications, communications and the human aspect of cybersecurity.

The methodology focuses primarily on how these systems, applications communicate, so it includes a methodology for:

1. Telecommunications (phones, VoIP, etc.)
2. Wired Networks
3. Wireless communications

#### Owasp
It's updated frequently and it's community-driven and it's solely used to test web application and services


#### NIST
is a popular framework used to improve an organisations cybersecurity standards and manage the risk of cyber threats.

The framework provides guidelines on security controls & benchmarks for success for organisations from critical infrastructure (power plants, etc.) all through to commercial.  There is a limited section on a standard guideline for the methodology as we penetration testers should take.


#### NCSC CAF

 is an extensive framework of fourteen principles used to assess the risk of various cyber threats and an organisation's defences against these.

  
  The framework applies to organisations considered to perform "vitally important services and activities" such as critical infrastructure, banking, and the likes. The framework mainly focuses on and assesses the following topics:

- Data security
- System security
- Identity and access control
- Resiliency
- Monitoring
- Response and recovery planning


### Boxes' Of Penetration Testing

- *==BLACK BOX==* - No Knowledge of the target we are testing, so we should spend a lot of time taking our time on Info. Gathering (Stage 1) and enumeration (Stage 2). 

- **==Grey Box==** - Partial Knowledge of the system, most popular way if testing. Grey-Box testing, the limited knowledge given saves time, and is often chosen for extremely well-hardened attack surfaces.
- **White Box** - is a low-level process usually done by a software developer who knows programming and application logic.


# Principle Of Security

## CIA Triad

The CIA triad is an information security model that is used in consideration throughout creating a security policy.

**Confidentiality**
Protection of data from unauthorized access and misuse. Basically it means to only give access to the intended person only.

**Integrity**
This element of CIA is to make sure information is kept accurate and consistent unless authorized changes are made. Again Basically it means to keep the data unaltered.

**Availability**
The data to be useful, it must be available and accessible by the user, like when it's needed it should be instantly recovered.


## Principles Of Privileges

It is vital to administrate and correctly define the various levels of access to an information technology system individuals require. 

The levels of access given to individuals are determined on two primary factors:

- The individual's role/function within the organisation
- The sensitivity of the information being stored on the system

Two key concepts are used to assign and manage the access rights of individuals: Privileged Identity Management (*===PIM===*) and Privileged Access Management (or *==PAM ==* for short).

**PIM** -> used to translate a **==user's==** role within an organisation into an access role on a system.
**PAM** -> is the management of the privileges a system's access role has, amongst other things.

## Security Models 

According to a security model, any system or piece of technology storing information is called an **Information system**. 

#### The Bell-La Padula Model

This model is used to achieve the confidentiality element of the CIA triad. 

The model works by granting access to pieces of data (called objects) on a strictly need to know basis. This model uses the rule **=="no write down, no read up".==** 

![Enlarged image](https://tryhackme-images.s3.amazonaws.com/user-uploads/5de96d9ca744773ea7ef8c00/room-content/0e6e5d9d80785fc287b4a67e1453b295.png)

It got its on Advantage and Disadvantage:

| Adv.                                         | Dis.                                                                         |
| -------------------------------------------- | ---------------------------------------------------------------------------- |
| Policies can be replicated in organisations. | Users can't read it but know it exists, its not confidential in that aspect. |
| Simple to implement.                         | It relies on a large amount of trust within the organisation.                |
#### Biba model

This model applies the rule to objects (data) and subjects (users) that can be summarised as "no write up, no read down". This rule means that subjects **can** create or write content to objects at or below their level but **can only** read the contents of objects above the subject's level.

## Threat Modelling & Incident Response

***Threat Modelling*** is the process of reviewing, improving, and testing the security protocols in place in an organisation's information technology infrastructure and services.

The threat modelling process is very similar to a risk assessment made in workplaces for employees and customers. The principles all return to:

- Preparation
- Identification
- Mitigations
- Review

It is, however, a complex process that needs constant review and discussion with a dedicated team. An effective threat model includes:

- Threat intelligence
- Asset identification
- Mitigation capabilities
- Risk assessment

To help with this, there are frameworks such as **STRIDE**

|   |   |
|---|---|
|**Principle**|**Description**|
|Spoofing|This principle requires you to authenticate requests and users accessing a system. Spoofing involves a malicious party falsely identifying itself as another.<br><br>Access keys (such as API keys) or signatures via encryption helps remediate this threat.|
|Tampering|By providing anti-tampering measures to a system or application, you help provide integrity to the data. Data that is accessed must be kept integral and accurate.<br><br>For example, shops use seals on food products.|
|Repudiation|This principle dictates the use of services such as logging of activity for a system or application to track.|
|Information Disclosure|Applications or services that handle information of multiple users need to be appropriately configured to only show information relevant to the owner.|
|Denial of Service|Applications and services use up system resources, these two things should have measures in place so that abuse of the application/service won't result in bringing the whole system down.|
|Elevation of Privilege|This is the worst-case scenario for an application or service. It means that a user was able to escalate their authorization to that of a higher level i.e. an administrator. This scenario often leads to further exploitation or information disclosure.|
A breach of security is known as an **incident**. 

An incident is responded to by a **C**omputer **S**ecurity **I**ncident **R**esponse **T**eam (**CSIRT**) which is prearranged group of employees with technical knowledge about the systems and/or current incident. To successfully solve an incident, these steps are often referred to as the six phases of Incident Response that takes place, listed in the table below:

  

|   |   |
|---|---|
|**Action**|**Description**|
|Preparation|Do we have the resources and plans in place to deal with the security incident?|
|Identification|Has the threat and the threat actor been correctly identified in order for us to respond to?|
|Containment|Can the threat/security incident be contained to prevent other systems or users from being impacted?|
|Eradication|Remove the active threat.|
|Recovery|Perform a full review of the impacted systems to return to business as usual operations.|
|Lessons Learned|What can be learnt from the incident? I.e. if it was due to a phishing email, employees should be trained better to detect phishing emails.|

# WEB Penetration Testing
## Web Appliocation Using Buil-in resource

View Source - Use your browser to view the human-readable source code of a website.
Inspector - Learn how to inspect page elements and make changes to view usually blocked content.
Debugger - Inspect and control the flow of a page's JavaScript
Network - See all the network requests a page makes.

## Content Discovery

During Content Discovery what we want to do is try to look for contents that are not intended for public to view. So we can discover contents mainly in three ways: 
1. Manually
2. Automated
3. OSINT  (Open-Source Intellegence)

   1.1 Manually Discovery (/Robots.txt)
    The robots.txt file is a document that tells search engines which pages they are and aren't allowed to show on their search engine results or ban specific search engines from crawling the website altogether. It can be common practice to restrict certain website areas so they aren't displayed in search engine results. These pages may be areas such as administration portals or files meant for the website's customers. This file gives us a great list of locations on the website that the owners don't want us to discover as penetration testers.

   1.2 Manually (/favicon.txt)
     The favicon is a small icon displayed in the browser's address bar or tab used for branding a website.






Sometimes when frameworks are used to build a website, a favicon that is part of the installation gets leftover, and if the website developer doesn't replace this with a custom one, this can give us a clue on what framework is in use. OWASP host a database of common framework icons that you can use to check against the targets favicon https://wiki.owasp.org/index.php/OWASP_favicon_database(opens in new tab). Once we know the framework stack, we can use external resources to discover more about it (see next section).



Practical Exercise:



On the AttackBox, open firefox and enter the url https://static-labs.tryhackme.cloud/sites/favicon/(opens in new tab) here you'll see a basic website with a note saying "Website coming soon...", if you look at your tabs you'll notice an icon that confirms this site is using a favicon.

Viewing the page source you'll see line six contains a link to the images/favicon.ico file. 

If you run the following command on the AttackBox, it will download the favicon and get its md5 hash value which you can then lookup on the
https://wiki.owasp.org/index.php/OWASP_favicon_database(opens in new tab).

curl
user@machine$ curl https://static-labs.tryhackme.cloud/sites/favicon/images/favicon.ico | md5sum
Note: This curl will fail on the AttackBox if you are a free user, in which case you should use a VM for this. If your hash ends with 427e then your curl failed, and you may need to try it again. You could also run this on Windows in Powershell as shown below.

**==PowerShell==**
PS C:\> curl https://static-labs.tryhackme.cloud/sites/favicon/images/favicon.ico -UseBasicParsing -o favicon.ico

PS C:\> Get-FileHash .\favicon.ico -Algorithm MD5 

1.3 Manually (Sitemap.xml)

Unlike the robots.txt file, which restricts what search engine crawlers can look at, the sitemap.xml file gives a list of every file the website owner wishes to be listed on a search engine. These can sometimes contain areas of the website that are a bit more difficult to navigate to or even list some old webpages that the current site no longer uses but are still working behind the scenes.

1.4 Manually HTTP Headers

When we make requests to the web server, the server returns various HTTP headers. These headers can sometimes contain useful information such as the webserver software and possibly the programming/scripting language in use. In the below example, we can see the webserver is NGINX version 1.18.0 and runs PHP version 7.4.3. Using this information, we could find vulnerable versions of software being used. Try running the below curl command against the web server, where the -v switch enables verbose mode, which will output the headers (there might be something interesting!).


curl
user@machine$ curl http://{website} -v

1.5 Manually Framework Stack

  Once you've established the framework of a website, either from the above favicon example or by looking for clues in the page source such as comments, copyright notices or credits, you can then locate the framework's website. From there, we can learn more about the software and other information, possibly leading to more content we can discover.

  3. OSINT

     Google Hacking / Dorking

Google hacking / Dorking utilizes Google's advanced search engine features, which allow you to pick out custom content. You can, for instance, pick out results from a certain domain name using the site: filter, for example (site:tryhackme.com) you can then match this up with certain search terms, say, for example, the word admin (site:tryhackme.com admin) this then would only return results from the tryhackme.com website which contain the word admin in its content. You can combine multiple filters as well. Here is an example of more filters you can use:

   Wapplayzer

  is an online tool and browser extension that helps identify what technologies a website uses, such as frameworks, Content Management Systems (CMS), payment processors and much more, and it can even find version numbers as well.

Wayback Machine

  is a historical archive of websites that dates back to the late 90s. You can search a domain name, and it will show you all the times the service scraped the web page and saved the contents. This service can help uncover old pages that may still be active on the current website.

  Github

  To understand GitHub, you first need to understand Git. Git is a version control system that tracks changes to files in a project. Working in a team is easier because you can see what each team member is editing and what changes they made to files. When users have finished making their changes, they commit them with a message and then push them back to a central location (repository) for the other users to then pull those changes to their local machines. GitHub is a hosted version of Git on the internet. Repositories can either be set to public or private and have various access controls. You can use GitHub's search feature to look for company names or website names to try and locate repositories belonging to your target. Once discovered, you may have access to source code, passwords or other content that you hadn't yet found.

  S3 Buckets

S3 Buckets are a storage service provided by Amazon AWS, allowing people to save files and even static website content in the cloud accessible over HTTP and HTTPS. The owner of the files can set access permissions to either make files public, private and even writable. Sometimes these access permissions are incorrectly set and inadvertently allow access to files that shouldn't be available to the public. The format of the S3 buckets is http(s)://{name}.s3.amazonaws.com(opens in new tab) where {name} is decided by the owner, such as tryhackme-assets.s3.amazonaws.com(opens in new tab). S3 buckets can be discovered in many ways, such as finding the URLs in the website's page source, GitHub repositories, or even automating the process. One common automation method is by using the company name followed by common terms such as {name}-assets, {name}-www, {name}-public, {name}-private, etc.

2. ## Automation

What is Automated Discovery?

Automated discovery is the process of using tools to discover content rather than doing it manually. This process is automated as it usually contains hundreds, thousands or even millions of requests to a web server. These requests check whether a file or directory exists on a website, giving us access to resources we didn't previously know existed. This process is made possible by using a resource called wordlists.



What are wordlists?

Wordlists are just text files that contain a long list of commonly used words; they can cover many different use cases. For example, a password wordlist would include the most frequently used passwords, whereas we're looking for content in our case, so we'd require a list containing the most commonly used directory and file names. An excellent resource for wordlists that is preinstalled on the THM AttackBox is https://github.com/danielmiessler/SecLists(opens in new tab) which Daniel Miessler curates.



Automation Tools

Although there are many different content discovery tools available, all with their features and flaws, we're going to cover three which are preinstalled on our attack box, ffuf, dirb and gobuster.



On the AttackBox execute the following three commands, targeting the Acme IT Support website and see what results you get.



Using ffuf:
ffuf
user@machine$ ffuf -w /usr/share/wordlists/{THE PLACE}/common.txt -u http://10.64.182.71/FUZZ

Using dirb:
dirb
user@machine$ dirb http://10.64.182.71/ /usr/share/wordlists/{THE PLACE/common.txt

Using Gobuster:
gobuster
user@machine$ gobuster dir --url http://MACHINE_IP/ -w /usr/share/wordlists/{THE PLACE}/common.txt

## Sub-Domain Enumeration

is the process of finding valid subdomains for a domain, but why do we do this? We do this to expand our attack surface to try and discover more potential points of vulnerability.

We will explore three different subdomain enumeration methods: 
1. Brute Force
2.  OSINT (Open-Source Intelligence) and
3.  Virtual Host.

### OSINT - SSL/TLS Certificates

When an SSL/TLS (Secure Sockets Layer/Transport Layer Security) certificate is created for a domain by a CA (Certificate Authority), CA's take part in what's called "Certificate Transparency (CT) logs". These are publicly accessible logs of every SSL/TLS certificate created for a domain name. The purpose of Certificate Transparency logs is to stop malicious and accidentally made certificates from being used. We can use this service to our advantage to discover subdomains belonging to a domain, sites like https://crt.sh(opens in new tab) offer a searchable database of certificates that shows current and historical results.

### OSINT - Search Engines

Search engines contain trillions of links to more than a billion websites, which can be an excellent resource for finding new subdomains. Using advanced search methods on websites like Google, such as the site: filter, can narrow the search results. For example, site:*.domain.com -site:www.domain.com would only contain results leading to the domain name domain.com but exclude any links to www.domain.com; therefore, it shows us only subdomain names belonging to domain.com.

## DNS Bruteforce

Bruteforce DNS (Domain Name System) enumeration is the method of trying tens, hundreds, thousands or even millions of different possible subdomains from a pre-defined list of commonly used subdomains. Because this method requires many requests, we automate it with tools to make the process quicker. In this instance, we are using a tool called **dnsrecon** to perform this.

## OSINT - Sublist3r

To speed up the process of OSINT subdomain discovery, we can automate the above methods with the help of tools like Sublist3r.

## Virtual Hosts


