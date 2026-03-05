
## Architecture Overview And High Level Design

## Architecture

This repository demonstrates setup process of an application that will use an agent on Amazon Bedrock

![Diagram](images/diagram.png)

## The solution consists of:

AWS S3 bucket which will be used to store the domain data that the Agent will later use to answer domain specifc questions

Amazon Bedrock Knowledge Base with S3 Data Source, S3 Vector Database, Titan Embedding Model

AWS Lambda function which serves as a backend API for the AI agent

Amazon Bedrock Agent and Action Group

EC2 Instance to run Streamlit UI


## High Level Design.

## Overview
In this project, we will set up an Amazon Bedrock agent with an action group that dynamically creates an investment company portfolio based on specific parameters. The agent also has Q&A capabilities for Federal Open Market Committee (FOMC) reports, leveraging a Streamlit framework for the user interface. 

This README will walk you through the step-by-step process to set up the Amazon Bedrock GenAI agent manually using the AWS Console.

## Setup

### Step 1: Creating S3 Buckets
- Please make sure that you are in the **us-west-2** region. If another region is required, you will need to update the region variable `theRegion` in the `invoke_agent.py` file code. 
- **Domain Data Bucket**: Create an S3 bucket to store the domain data. For example, call the S3 bucket `knowledgebase-bedrock-agent-{alias}`. We will use the default settings.

Give {alias} in bucket name as some unique text

Once bucket create successfully. Upload the pdf files from S3docs directory in this repository to S3 bucket.
These files are the Federal Open Market Committee documents describing monetary policy decisions made at the Federal Reserved board meetings. The documents include discussions of economic conditions, policy directives to the Federal Reserve Bank of New York for open market operations, and votes on the federal funds rate.

### Step 2: Knowledge Base Setup in Bedrock Agent

Please make sure you are not logged in as root user to create KB. If yes, create a admin user in IAM and then proceed with below steps to create KB.

Open Amazon Bedrock and Navigate to Knowledge Bases under Build

![KB](images/KB.png)

selecting the orange button **Create knowledge base** with dropdown option **Knowledge Base with Vector Store**

![CreateKB](images/Create-KB.png)

You can use the default name, or enter in your own. Then, select **Next** at the bottom right of the screen keeping all default values as it is

Configure S3 database as Data Source by providing S3 Uri as S3 bucket created to store pdf data. Rest Keep Default and press Next

For the embedding model, choose **Amazon: Titan Embeddings G1 - Text**. Choose Vector Store type as **Amazon S3 Vectors** and scroll down to select **Next**.

![Embd](images/Embeddings-model.png)

- On the next screen, review your work, then select **Create knowledge base**
(Creating the knowledge base may take a few minutes. Please wait for it to finish before going to the next step.)

Once your Knowledge Base is created successfully. you can that listed in Konwledge bases as below

 ![Kb-done](images/KB-done.png)
 
### Step 3: Lambda Function Configuration
- Create a Lambda function (Python 3.14) for the Bedrock agent's action group. We will call this Lambda function `PortfolioCreator-actions`.

![Lambda](images/Lambda-Create.png)

- Copy the python code from the file **ActionLambda.py** into your Lambda function.

- Then, select **Deploy** in the tab section of the Lambda console.

- Next, apply a resource policy to the Lambda to grant Bedrock agent access. To do this, we will switch the top tab from **code** to **configuration** and the side tab to **Permissions**. Then, scroll to the **Resource-based policy statements** section and click the **Add permissions** button.

![Permissions config](images/permissions_config.png)

![Lambda resource policy create](images/lambda_resource_policy_create.png)

- Select ***AWS service***, then use the following settings to configure the resource based policy:

* ***Service*** - `Other`
* ***Statement ID*** - `allow-bedrock-agent`
* ***Principal*** - `bedrock.amazonaws.com`
* ***Source ARN*** - `arn:aws:bedrock:us-west-2:{account-id}:agent/*` - (Please note, AWS recommends least privilage so only an allowed agent can invoke this Lambda function. A * at the end of the ARN grants any agent in the account access to invoke this Lambda. Ideally, we would not use this in a production environment.)
* ***Action*** - `lambda:InvokeFunction`

![Lambda resource policy](images/lambda_resource_policy.png)

- Once your configurations look similar to the above screenshot, select ***Save*** at the bottom.


### Step 4: Setup Bedrock Agent and Action Group 

- Navigate to the Bedrock console. Go to the toggle on the left, and under ***Builds*** select ***Agents***. Provide an agent name, like ***PortfolioCreator*** then create the agent.

- The agent description is optional, and we will use the default new service role. For the model, select **Anthropic: Claude 3 Haiku**. Next, provide the following instruction for the agent

```instruction
You are an investment analyst. Your job is to assist in investment analysis, create research summaries, generate profitable company portfolios, and facilitate communication through emails. Here is how I want you to think step by step:

1. Portfolio Creation:
    Analyze the user's request to extract key information such as the desired number of companies and industry. 
    Based on the criteria from the request, create a portfolio of companies. Use the template provided to format the portfolio.

2. Company Research and Document Summarization:
    For each company in the portfolio, conduct detailed research to gather relevant financial and operational data.
    When a document, like the FOMC report, is mentioned, retrieve the document and provide a concise summary.

3. Email Communication:
    Using the email template provided, format an email that includes the newly created company portfolio and any summaries of important documents.
    Utilize the provided tools to send an email upon request, That includes a summary of provided responses and portfolios created.
```

![Agent](images/Agent.png)

- After, scroll to the top and **Save**

- The instructions for the Generative AI Investment Analyst Tool outlines a comprehensive framework designed to assist in investment analysis. This tool is tasked with creating tailored portfolios of companies based on specific industry criteria, conducting thorough research on these companies, and summarizing relevant financial documents. Additionally, the tool formats and sends professional emails containing the portfolios and document summaries. The process involves continuous adaptation to user feedback and maintaining a contextual understanding of ongoing requests to ensure accurate and efficient responses.
