# FinOps Baseline

Cluster: finops-lab

Applications:
- Nginx
- Redis
- MySQL
- Java App
- Stress Generator

Observations:
- CPU requests heavily exceed actual usage
- Memory requests heavily exceed actual usage
- Significant overprovisioning detected

Optimization Target:
- Reduce CPU requests
- Reduce Memory requests
- Implement HPA
- Validate node resiliency

Expected Savings:
40-60%