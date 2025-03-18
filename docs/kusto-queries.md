# Useful Kusto queries to inspect tools calls

This document provides guidance on how to monitor the performance of AI agents within the application using Kusto queries.

### Importance of Monitoring AI Agent Performance
Monitoring the performance of AI agents is crucial for ensuring they operate efficiently and effectively. By analyzing requests and responses, you can identify potential issues, optimize performance, and enhance user experience.

### Example Query Results
Here are some example results you might see when running the Kusto queries:

- **Inspect all Azure OpenAI responses**: This query will return the response details from the OpenAI service, including the content of the response and any associated metadata.
- **Inspect all Azure OpenAI requests**: This query will show the details of the requests made to the OpenAI service, including the request body and headers.

### Key Performance Metrics
When monitoring AI agent performance, consider tracking the following metrics:
- **Response Time**: Measure how long it takes for the AI agent to respond to user queries.
- **Error Rate**: Track the number of failed requests or errors returned by the AI agent.
- **Throughput**: Monitor the number of requests processed by the AI agent over a specific time period.

### Troubleshooting Tips
If you encounter issues while monitoring AI agent performance, consider the following tips:
- Ensure that Application Insights is properly configured and enabled in your application.
- Check the Kusto query syntax for any errors or typos.
- Review the logs in Application Insights for any error messages or warnings that may indicate issues with the AI agent.

### Additional Resources
For more information on Kusto queries and performance monitoring, check out the following resources:
- [Kusto Query Language Documentation](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/)
- [Application Insights Documentation](https://docs.microsoft.com/en-us/azure/azure-monitor/app/app-insights-overview)

### User Feedback
We encourage users to provide feedback on their experiences with monitoring AI agent performance. Your insights can help improve the documentation and the overall project. You can reach us at feedback@example.com or fill out our [feedback form](https://example.com/feedback).

### Visual Aids
This document will include diagrams or flowcharts to illustrate how Kusto queries fit into the overall monitoring process.

### Real-World Use Cases
This section will include specific scenarios where monitoring AI agent performance has led to significant improvements, providing practical insights for users.

### Expanded FAQs Section
This section will address common questions related to performance monitoring, such as how to interpret specific metrics or what to do if performance is below expectations.

### Best Practices
This section will provide best practices for monitoring AI agents, including tips on setting up alerts, regular review of performance metrics, and optimizing queries for better performance.



Monitoring the performance of AI agents is crucial for ensuring they operate efficiently and effectively. By analyzing requests and responses, you can identify potential issues, optimize performance, and enhance user experience.


By default all Azure Open AI requests and response are logged in the java application logs.
The following Kusto queries can be used to inspect the tools calls requests and responses.


### Inspect all Azure OpenAI responses

```kusto
traces
| where cloud_RoleName == "copilot-api"
| where message contains "openai.azure.com/openai/deployments"
| extend chatmessage=parse_json(message)
| where chatmessage["az.sdk.message"] == "HTTP response"
| extend chatbody=parse_json(tostring(chatmessage.body)).choices[0]
```
### Inspect all Azure OpenAI requests

```kusto
traces 
| where cloud_RoleName == "copilot-api"
| where message contains "openai.azure.com/openai/deployments"
| extend chatrequest=parse_json(message)
| where chatrequest.method == "POST" 
| extend chatbody=parse_json(tostring(parse_json(message).body))
```

### Inspect all tools calls requests

```kusto
traces
| where cloud_RoleName == "copilot-api"
| where message contains "openai.azure.com/openai/deployments"
| extend chatrequest=parse_json(message)
| where chatrequest["az.sdk.message"] == "HTTP response"
| extend response=parse_json(tostring(parse_json(tostring(chatrequest.body)))).choices[0]
| where response.finish_reason=="tool_calls"
| extend tool_calls=parse_json(tostring(parse_json(tostring(response.message)).tool_calls))
```

### Inspect all tools calls requests for TransactionHistoryPlugin function

```kusto
traces 
| where cloud_RoleName == "copilot-api"
| where message contains "openai.azure.com/openai/deployments"
| extend chatrequest=parse_json(message)
| where chatrequest["az.sdk.message"] == "HTTP response" 
| extend response=parse_json(tostring(parse_json(tostring(chatrequest.body)))).choices[0]
| where response.finish_reason=="tool_calls" //and response.message contains "TransactionHistoryPlugin"
| extend tool_calls=parse_json(tostring(parse_json(tostring(response.message)).tool_calls))
```
