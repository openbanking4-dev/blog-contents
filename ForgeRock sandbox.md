# ForgeRock Open Banking Reference Implementation

For illustrate how Open Banking works, we are going to use the ForgeRock model bank. It's a sandbox environment available online, for free and accessible to anyone.
ForgeRock has work with OBIE, the author of this standard, on building this instance. The idea is really to democratise Open Banking and help innovation in the financial sector.
Special thanks for those two companies, to have take in charge the cost of developing and hosting this instance, just so we can learn Open Banking for free.
You probably notice that I work for ForgeRock and therefore those set of tutorials are sponsored by them. Once again, it's in line with the idea of democratising Open Banking.
In this article, I would like to take the time to present the ForgeRock model bank that we are going to use.

## What is ForgeRock

ForgeRock helps people safely and simply access the connected world. It's an identity provider that works actively in securing access, building trust model and encourage data sharing in a secure and control manner for the user.
There is no wonder why they are in the Open Banking space: Open Banking shares the same values than ForgeRock.
From a business point of view, they principally provides software for companies with identity needs, especially the one that wishes to provide secured APIs. In the Open Banking space, it's mainly banks that ForgeRock will help building Open Banking solution.

## The ForgeRock Open Banking eco-system

You can't provide a sandbox without providing a set of tools. We are going to talk about here of how the sandbox is composed.
There is no secret, ForgeRock behind is selling Open Banking solution, which was used to build this sandbox.
Even if we are just going to consume those APIs, I thought it would be good to gives some insight of what is behind.

### ForgeRock Open Banking directory

As we mention in a previous article, there is a trust model in Open Banking. The issue with it is that for a sandbox purposes, you kind of want to simplify the access to the APIs.
You could say: why not making the APIs just less secured so anyone can use it? 
A part of what is interesting with the Open Banking standard is the security around it. Providing APIs in a secured way, between three parties (you, the user and the bank) is a real challenge.
The goal of a model bank like ForgeRock is to democratise the security relative to Open Banking.
This is why ForgeRock took the approach of creating an Open Banking directory, accessible to anyone and without background check.
You basically registered and you are part of the Open Banking circle of trust for the ForgeRock sandbox.
In a way, the security is less restrictive by how you can access the directory. In the production environment, getting access to the directory is controlled, a fairly manual process that is unfortunately slow at the moment. 
For the ForgeRock directory, you create an account and job is done, you got access to the directory.

The directory we are going to use is:
https://directory.ob.forgerock.financial/

For people interested to build their own Open Banking eco-system, be aware that you can buy the software and host your own instance of the directory.


### The JWKMS

One thing that you will see very soon once we start using the APIs, is that it is based on certificates, signing messages.
Which is quite cool, although as a developer, you need to be able to do a bit of cryptography on your side.
By experience, it's probably the part the most disturbing for developers when you first discover the Open Banking standard.
As the idea is to help you learn Open Banking, ForgeRock provided a tool called the JWKMS.
Basically it's an application, available online via APIs, that will handle the cryptography for you.
It's quite handy because you don't need to worry from the beginning about it. The idea is that you learn how works Open Banking and when the time comes, you start implementing your own JWKMS.

JWKMS stands for JSON Web Key Management Services. It's all about cryptography but around the JOSE standard, which is a standard used by Open Banking underneath.
We are going to manipulate JWTs (JSON Web Tokens), provides our public keys as JWKs (JSON Web Keys), so it's why it's called JWKMS.

As all the application we are going to present, the JWKMS is part of the Open Banking solution offered by ForgeRock, that you can buy if you are interested.


### The ForgeRock bank

The star of the show, the ForgeRock bank exposes Open Banking APIs. This is the application that we are going to consume the APIs and learn how Open Banking works.
It implements all the versions of the standards, and support all the Open Banking APIs. Because yes, some are optionals.
It's also supporting dynamic registration, that we are going to use for on-boarding.
There is also so interesting features for us, like headless auth. It's basically a way to bypass the user interaction and focussing on consuming the APIs. Obviously, it only make sense in a sandbox context, although it's quite nice for a developer to be able to automatise his tests without having to mock the UI interaction.
The last fact about the ForgeRock bank is the security conformance. It's really interesting for the developer to know that the ForgeRock bank has passed the different security conformance and is FAPI certified, and this since the beginning of Open Banking. It means that you are going to learn against a standardise and stable APIs, which is quite important in our learning journey.

You would need to create a user to this bank, the link is here
https://bank.ob.forgerock.financial/register
I will repeat this in due time, don't worry.

From an API point of view, we are going to be more interested in knowing the different endpoints.
Fortunately, you don't need to get them one by one, there are two endpoints dedicated into discovering the services.

For the resource server APIs:
```
https://rs.aspsp.ob.forgerock.financial/open-banking/discovery
```

For the authorisation server APIs:
```
https://as.aspsp.ob.forgerock.financial/oauth2/.well-known/openid-configuration
```

### The Open Banking Analytics

A bit more atypic and less of a use for us as developer, the ForgeRock offers an Open Banking analytics. It's basically an application, in the same line than google analytics, dedicated in monitoring Open Banking KPIs. 
ForgeRock hasn't provide any report about the usage of their sandbox yet, despite it would be interesting for the community to get an idea of how popular Open Banking is getting by looking at the statistic of the sandbox.

### ForgeRock Open Banking Postman collection

We do provide a postman collection to play with. It's actually would be our base tool for this tutorial.
It's available here:
https://postman.ob.forgerock.financial

On the top right, there is a button that should automatically launch postman and download the collection/environment into it.

We choose postman as it is language agnostic, and the tools to go for playing with APIs those days.

You can already download it now if you are interested to follow this tutorial.


## Conclusion

The ForgeRock model bank is going to be a good learning tool for us. It was important to understand who is ForgeRock, what is their role in the Open Banking space and why they do provide a free service like this.
I still think it's awesome for developers to have a free service, certified so of high quality, that you can rely to learn Open Banking.

