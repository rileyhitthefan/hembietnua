---
title: "Lab setup"
date: 2024-06-27T14:35:46+07:00
draft: false
weight: 3
---

## Download and configure the assets for the labs
1. In the AWS Cloud9 IDE, select the **bash terminal**.

![Bash](/images/2-Bedrock/prep/Prep-16.png)

2. Paste and run the following into the terminal to download and unzip the code.

```
cd ~/environment/
curl 'https://static.us-east-1.prod.workshops.aws/public/4aaab724-21a5-4155-80ee-db443f910e6c/assets/workshop.zip' --output workshop.zip
unzip workshop.zip
```

Once completed, you should see the unzip results in the terminal:

![Bash](/images/2-Bedrock/prep/Prep-17.png)

3. Install the dependencies for the labs.
```
pip3 install -r ~/environment/workshop/setup/requirements.txt -U
```

If everything worked properly, you should see a success message (you can disregard the notice like below):

![Bash](/images/2-Bedrock/prep/Prep-18.png)

4. Verify configuration by pasting and running the following into the AWS Cloud9 terminal:

```
cd ~/environment/
python ~/environment/workshop/completed/api/bedrock_api.py
```

If everything is working properly, you should see a response about Manchester, New Hampshire:

![Bash](/images/2-Bedrock/prep/Prep-19.png)

## You're ready to try some labs!
Any of the following sections could be a good place to start, depending on your experience level and interests:

- [🧱Foundational concepts Labs](../2.2-foundational/)
- [📙Basic patterns Labs](../2.3-basic/)
- [📚Text patterns Labs](../2.4-text/)
- [🌃Multimodal & Image Labs](../2.5-image/)
- [🔐Security & Safeguard Labs](../2.6-security/)
- [💬Prompt Engineering Labs](../2.7-prompteng/)
- [🎁Bonus Labs](../2.8-bonus/)

