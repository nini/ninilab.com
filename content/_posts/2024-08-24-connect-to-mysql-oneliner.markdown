---
title: "Streamline MySQL Access in Kubernetes"
date: 2024-08-24 14:28:00 +0200
excerpt: Learn how to automate MySQL configuration in Kubernetes using JSON files and a one-liner script.
categories:
  - DevOps
  - Kubernetes
tags:
  - Kubernetes
  - MySQL
  - JSON
  - Automation
  - Scripting
splash_image: /assets/images/2024-08-24-mysql-config.jpg
---

Today I encountered a classic problem in cloud environments—accessing MySQL in a Kubernetes setup where direct connections are restricted by VPC policies. MySQL was only accessible through the application, making it difficult to manage configurations manually. I needed a way to dynamically generate MySQL configurations from JSON files stored within the Kubernetes environment.

## The Problem

In a secured cloud environment, accessing MySQL directly from a local machine was impossible due to VPC restrictions. The only way to connect to the database was via the running application inside a Kubernetes pod. The MySQL connection details were stored in a JSON configuration file within the pod. This setup required transforming the JSON data into a MySQL configuration file and running MySQL commands from within the pod.

### Example Structure of the Application Configuration File

Let’s assume you have a JSON configuration file located at `config/config.staging.json` inside your pod. The structure might look like this:

```json
{
    "mysql": {
        "pools": {
            "myapp": {
                "user": "root",
                "password": "root",
                "host": "127.0.0.1",
                "port": 3306,
                "database": "myapp"
            }
        }
    }
}
```

This file contains all the necessary information to connect to your MySQL database, but it’s in a JSON format that MySQL doesn’t understand directly.

## The Solution: Automating with a One-Liner

To solve this, I crafted a one-liner that does everything—from processing the JSON file to generating a MySQL configuration and executing the necessary MySQL commands:

```bash
kubectl exec -it deployment/pod-example-123456 -- bash -c "apk add jq && cat config/config.staging.json | jq '.mysql.pools' | jq -r 'def kv: to_entries[] | \"\(.key)=\(.value)\"; [to_entries[] | \"[\(.key)]\", (.value|kv)]| join(\"\n\")' | sed -e 's/\[myapp\]/[client]/g' > mysql.conf ; mysql --defaults-file=mysql.conf"
```

This command achieves the following:
- **Installs `jq`** to process JSON files.
- **Extracts the necessary MySQL configuration** from the JSON file stored inside the Kubernetes pod.
- **Transforms the JSON data** into a format that MySQL can use by creating a `mysql.conf` file.
- **Executes MySQL commands** using the dynamically generated configuration file.

The process begins by installing `jq`, then reading the JSON configuration and extracting the `.mysql.pools` section. The JSON data is converted into key-value pairs suitable for a MySQL configuration file, with the `[myapp]` section header replaced by `[client]` to comply with MySQL’s configuration format. Finally, the MySQL client runs using the `mysql.conf` file.

## Why This Approach?

This method is particularly effective in Kubernetes environments where you can’t directly access MySQL. By automating the configuration generation, you save time and reduce the risk of errors that could occur with manual configuration management. This one-liner is a powerful tool in environments where automation is key, and it allows you to adapt quickly to restricted access setups.

## Conclusion

In today’s cloud-based infrastructures, especially within Kubernetes, you’re often faced with challenges like restricted database access. This one-liner script not only simplifies the process of managing MySQL configurations in such environments but also highlights the power of automation in DevOps.

By converting JSON configuration data into a MySQL-compatible format on the fly, you can seamlessly connect to databases even in the most restrictive environments. This approach ensures that you can focus on development and deployment without worrying about the underlying infrastructure limitations.

If you found this solution helpful, consider implementing it in your own workflows. Automation like this can significantly streamline your processes, making your life as a DevOps engineer much easier.

Happy coding, and stay tuned for more tips and tricks!
