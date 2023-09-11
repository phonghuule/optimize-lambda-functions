# **Optimize Lambda Function For Cost and Performance**

This lab is provided as part of **[AWS Innovate For Every Application Edition](https://aws.amazon.com/events/aws-innovate/apj/for-every-app/)**, it has been adapted from this [workshop](https://catalog.workshops.aws/serverless-optimization/en-US)

ℹ️ You will run this lab in your own AWS account. Please follow directions at the end of the lab to remove resources to avoid future costs.

## Introduction

In this lab, you will learn how to understand and gain insight into AWS Lambda functions during development. Specifically understanding how to configure resources for a given Lambda function to optimize for cost and performance. 
The goal is to understand how the execution time will change by increasing or decreasing the memory allocated to the Lambda function.

This lab will be using the **Lambda Power Tuning Project** by **Alex Casalboni**. This project is located in the [Serverless Application Repository](https://serverlessrepo.aws.amazon.com/applications/us-east-1/451282441545/aws-lambda-power-tuning).

![](/Images/power-tuning-visualization.png)

The skills you learn will help you build well-architected serverless applications in alignment with the [Serverless Applications Lens - AWS Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/serverless-applications-lens/cost-optimization.html)


## Deploy The Lambda Power Tuning Tool
The Lambda Power Tuning Tool is hosted in the [Serverless Application Repository (SAR)](https://aws.amazon.com/serverless/serverlessrepo/) 

SAR allows customers to quickly publish and deploy serverless applications. Customers can configure SAR permissions to allow access to specific accounts, accounts within Organizations, or public access for the ability to launch the application.

### 1. Log into the AWS console
Sign in to the AWS Management Console as an IAM user who has PowerUserAccess or AdministratorAccess permissions, to ensure successful execution of this lab.

### 2. Deploy The Lambda Power Tuning Tool
1. Browse to the [Serverless Application Repository](https://console.aws.amazon.com/serverlessrepo).

2. In the left-hand pane, make sure Available Applications is selected.

![](/Images/sar-left-pane.png)

3. In the filter window, type in **AWS Lambda Power Tuning**, and make sure **Show apps that create custom IAM roles or resource policies** is checked.

![](/Images/sar-filter.png)

4. Select the **aws-lambda-power-tuning application**.

![](/Images/power-tuning-search-page.png)

5. On the **Review, configure and deploy** page, leave the defaults for everything. Scroll to the bottom, check the **I acknowledge that this app creates custom IAM roles** option and click on **Deploy**

![](/Images/sar-deploy.png)

6. Browse to the [CloudFormation Console](https://console.aws.amazon.com/cloudformation) 
to review the installation of the power tuning tool. This will take several minutes. Once the stack is complete you can move on to the next section.

![](/Images/power-tune-complete.png)

## Deploy Test Lambda Function

Now that you have the power tuning tool installed, the next step is to run an AWS Lambda function against it. In this section of the module, you will be deploying a Lambda function to test the usage of the Power Tuning Tool. This function is memory intensive, using matrices.

### Deploy the function
ℹ️ Be sure to deploy this function in the same region as your Lambda Power Tuning Tool
1. Browse to the [AWS Lambda Console](https://console.aws.amazon.com/lambda) 
2. Click on the **Create Function** option on the upper right-hand side of the console.
![](/Images/power-tuning-create-function.png)
3. On the **Create function** page input the following values, leave the rest at their default values and click **Create Function**.
- Author from scratch: selected
- Function Name: ```lambda-power-tuning-test```
- Runtime: Python 3.8
- Architecture: x86_64

![](/Images/create-function.png)

4. In the Lambda Designer pane, click on **Layers** or alternatively scroll to the bottom of the page, to the Layers pane.

![](/Images/function-detail.png)

5. Click on the **Add Layer** button.

![](/Images/add-layer.png)

6. On the **Add layer** page input the following values and click the **Add** button.
- Layer Source: AWS Layers selected
- AWS layers: AWSLambda-Python38-SciPy1x selected
- Version: latest version available selected

![](/Images/layer-detail.png)

7. Locate the **Function Code** pane and paste the following code into the editor screen.

![](/Images/power-tuning-add-function-code.png)

```
import json
import numpy as np
from scipy.spatial import ConvexHull

def lambda_handler(event, context):

    ms = 100
    print("Printing from version on 10302020 - size of matrix", ms,"x",ms)

    print("\nFilling the matrix with random integers below 100\n")

    matrix_a = np.random.randint(100, size=(ms, ms))
    print(matrix_a)

    print("random matrix_b =")
    matrix_b = np.random.randint(100, size=(ms, ms))
    print(matrix_b)

    print("matrix_a * matrix_b = ")
    print(matrix_a.dot(matrix_b))

    num_points = 10
    print(num_points, "random points:")
    points = np.random.rand(num_points, 2)
    for i, point in enumerate(points):
        print(i, '->', point)

    hull = ConvexHull(points)
    print("The smallest convex set containing all",
        num_points, "points has", len(hull.simplices),
        "sides,\nconnecting points:")
    for simplex in hull.simplices:
        print(simplex[0], '<->', simplex[1])

    return {
        'statusCode': 200,
        'headers': {
            'Content-Type': 'application/json',
            'Access-Control-Allow-Origin': '*'
        },
        'body': '',
        "isBase64Encoded": False
    }
```

8. Click **Deploy** to deploy the new code for the function

![](/Images/function-edit.png)

9. In the function menu, click the **Configuration** option.
![](/Images/power-tuning-configuration-tab.png)

10. Click on **Edit** in the **General Configuration pane**.
![](/Images/power-tuning-general-config-edit.png)

11. On the **Edit basic settings** page input the following values, leave the remaining at their default and click the **Save** button:
- Memory: 3008
- Timeout: 5 minutes

![](/Images/function-settings.png)

### Test the function
1. On the function menu, select the **Test** tab. Input a name value and click the Test button to invoke a test of the function.
- Name: testevent

![](/Images/power-tuning-create-test.png)

2. After the function has executed. You will see a success message on the screen.

![](/Images/function-run.png)

3. Scroll to the top of the Function page to copy the function ARN, you will need this value for the next portion of the lab.

![](/Images/function-run.png)

## Analysis

Now that you have deployed the power tuning tool and a test Lambda function, it's time to find the best memory size for this function. The Power Tuning Tool will run our test function with various memory sizes to help determine which configuration provides the best cost and performance. In this section we'll walk through the steps to analyze our function's configuration.

1. In a new browser tab, open the [Step Function console](https://console.aws.amazon.com/states/) 

2. In the left-hand pane, make sure **State Machines** is selected.
![](/Images/step-left-pane.png)

3. Select the **powerTuningStateMachine** from the **State Machines** list.

![](/Images/state-machines.png)

4. In the newly opened powerTuningStateMachine page click the Start Execution button.

![](/Images/state-machine-detail.png)

5. A **Start execution** window will open. Replace the information in the **Input** field with the json below. Make sure to replace the value of the **lambdaARN** field with the ARN you copied in the previous stage. Then click the Start execution button.

```
{
	"lambdaARN": "YOUR LAMBDA ARN HERE",
	"powerValues": [128, 256, 512, 1024, 2048, 3008],
	"num": 10,
	"payload": "{}",
	"parallelInvocation": true,
	"strategy": "cost"
}
```

![](/Images/execution-info.png)

6. Wait for the execution to complete. Once the **Execution Status** shows Succeeded, click on the **Execution Output** menu tab

![](/Images/execution-details.png)

7. Copy the URL from the visualization field and paste it into a new browser tab to review the results.

![](/Images/execution-output.png)

8. This visualization gives you both recommendations and a visual output of the test results from the Power Tuning Tool.
![](/Images/power-tuning-visualization.png)

From the analysis we can conclude that to optimize the function for the lowest cost a memory configuration of 512MB would be best. To optimize the function for performance (i.e. the lowest invocation time) a memory configuration of 2048MB would be best.

If the function was originally configured with 128MB then increasing the memory to 512MB can reduce the cost AND improve the performance. Increasing the memory above 2048MB does not reduce the invocation time and therefore only leads to high costs. This shows the importance of optimizing Lambda functions within your environment.

This lab illustrates the cost and performance tradeoff when designing and executing lambda functions. The Lambda Tuning application can be used to gauge the resource requirements and analyze the cost profile. The Power Tuning application will be used in the later Graviton module to analyze a function performance on various compute architectures.

## Tear Down

The following instructions will remove the resources that you have created in this lab.

### Delete Lambda Power Tuning Tool
1. Navigate to the [CloudFormation Console](https://console.aws.amazon.com/cloudformation) page
2. Search for the **aws-lambda-power-tuning** stack. Select the stack and then click the **Delete** button.

![](/Images/power-tuning-delete.png)

This delete will take several minutes. Once the delete is complete you are free to move on to the next section.)

### Delete test Lambda Function
1. Navigate to the [Lambda Console](https://console.aws.amazon.com/lambda) 
2. Select **Functions** on the left-hand menu. Search for **lambda-power-tuning-test** function, select it and click **Actions > Delete**

![](/Images/power-tuning-lambda-delete.png)
