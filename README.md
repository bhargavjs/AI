🚀 WORKFLOW 1 — Docker Container Health Monitor

Checks containers every 5 minutes

If a container is “exited”, it restarts it

Sends notification (Telegram / Email optional)


JSON

{
  "name": "Docker Container Health Monitor",
  "nodes": [
    {
      "id": "Cron1",
      "name": "Cron",
      "type": "n8n-nodes-base.cron",
      "typeVersion": 1,
      "position": [300, 300],
      "parameters": {
        "triggerTimes": {
          "item": [
            {
              "interval": [
                {
                  "field": "minutes",
                  "value": 5
                }
              ]
            }
          ]
        }
      }
    },
    {
      "id": "Exec1",
      "name": "List Containers",
      "type": "n8n-nodes-base.executeCommand",
      "typeVersion": 1,
      "position": [600, 300],
      "parameters": {
        "command": "docker ps -a --format \"{{.Names}} {{.Status}}\""
      }
    },
    {
      "id": "IF1",
      "name": "Filter Stopped Containers",
      "type": "n8n-nodes-base.if",
      "typeVersion": 1,
      "position": [900, 300],
      "parameters": {
        "conditions": {
          "string": [
            {
              "value1": "={{$json[\"stdout\"]}}",
              "operation": "contains",
              "value2": "Exited"
            }
          ]
        }
      }
    },
    {
      "id": "Exec2",
      "name": "Restart Container",
      "type": "n8n-nodes-base.executeCommand",
      "typeVersion": 1,
      "position": [1200, 200],
      "parameters": {
        "command": "docker restart syn-backend"
      }
    }
  ],
  "connections": {
    "Cron1": {
      "main": [
        [
          {
            "node": "List Containers",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "List Containers": {
      "main": [
        [
          {
            "node": "Filter Stopped Containers",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Filter Stopped Containers": {
      "main": [
        [
          {
            "node": "Restart Container",
            "type": "main",
            "index": 0
          }
        ]
      ]
    }
  }
}









🚀 WORKFLOW 2 — Auto Deployment from GitHub (Webhook Trigger)

After push to main branch → n8n pulls code → rebuilds docker → restarts container.

JSON


{
  "name": "Auto Deployment from GitHub",
  "nodes": [
    {
      "id": "Webhook1",
      "name": "GitHub Webhook",
      "type": "n8n-nodes-base.webhook",
      "typeVersion": 1,
      "position": [300, 300],
      "parameters": {
        "httpMethod": "POST",
        "path": "deploy-app"
      }
    },
    {
      "id": "Exec3",
      "name": "Pull Code",
      "type": "n8n-nodes-base.executeCommand",
      "typeVersion": 1,
      "position": [600, 300],
      "parameters": {
        "command": "cd /home/syn/app && git pull origin main"
      }
    },
    {
      "id": "Exec4",
      "name": "Rebuild Docker",
      "type": "n8n-nodes-base.executeCommand",
      "typeVersion": 1,
      "position": [900, 300],
      "parameters": {
        "command": "cd /home/syn/app && docker compose down && docker compose up -d --build"
      }
    }
  ],
  "connections": {
    "GitHub Webhook": {
      "main": [
        [
          {
            "node": "Pull Code",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Pull Code": {
      "main": [
        [
          {
            "node": "Rebuild Docker",
            "type": "main",
            "index": 0
          }
        ]
      ]
    }
  }
}







🚀 WORKFLOW 3 — Daily Database Backup to Google Drive or S3
JSON

{
  "name": "Daily DB Backup",
  "nodes": [
    {
      "id": "CronBackup",
      "name": "Cron",
      "type": "n8n-nodes-base.cron",
      "typeVersion": 1,
      "position": [300, 300],
      "parameters": {
        "triggerTimes": {
          "item": [
            {
              "hour": 2,
              "minute": 0
            }
          ]
        }
      }
    },
    {
      "id": "DumpDB",
      "name": "Dump Database",
      "type": "n8n-nodes-base.executeCommand",
      "typeVersion": 1,
      "position": [600, 300],
      "parameters": {
        "command": "mongodump --out /home/backups/db-{{Date.now()}}"
      }
    },
    {
      "id": "UploadS3",
      "name": "Upload to S3",
      "type": "n8n-nodes-base.awsS3",
      "typeVersion": 1,
      "position": [900, 300],
      "parameters": {
        "operation": "upload",
        "bucketName": "syn-backups",
        "fileName": "backup-{{Date.now()}}.zip",
        "filePath": "/home/backups/"
      }
    }
  ],
  "connections": {
    "Cron": {
      "main": [
        [
          {
            "node": "Dump Database",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Dump Database": {
      "main": [
        [
          {
            "node": "Upload to S3",
            "type": "main",
            "index": 0
          }
        ]
      ]
    }
  }
}





🚀 WORKFLOW 4 — VPS CPU / RAM / Disk Monitoring
JSON


{
  "name": "VPS Resource Monitor",
  "nodes": [
    {
      "id": "Cron5",
      "name": "Cron",
      "type": "n8n-nodes-base.cron",
      "typeVersion": 1,
      "position": [300, 300],
      "parameters": {
        "triggerTimes": {
          "item": [
            {
              "interval": [
                {
                  "field": "minutes",
                  "value": 1
                }
              ]
            }
          ]
        }
      }
    },
    {
      "id": "Exec6",
      "name": "Check Resources",
      "type": "n8n-nodes-base.executeCommand",
      "typeVersion": 1,
      "position": [600, 300],
      "parameters": {
        "command": "echo CPU:$(top -bn1 | grep 'Cpu(s)' | awk '{print $2+$4}') RAM:$(free -m | awk 'NR==2 {print $3}') DISK:$(df -h / | awk 'NR==2 {print $5}')"
      }
    }
  ],
  "connections": {
    "Cron": {
      "main": [
        [
          {
            "node": "Check Resources",
            "type": "main",
            "index": 0
          }
        ]
      ]
    }
  }
}



🚀 WORKFLOW 5 — SSL Expiry Alert
JSON


{
  "name": "SSL Expiry Alert",
  "nodes": [
    {
      "id": "CronSSL",
      "name": "Daily Check",
      "type": "n8n-nodes-base.cron",
      "typeVersion": 1,
      "position": [300, 300],
      "parameters": {
        "triggerTimes": {
          "item": [
            {
              "hour": 8,
              "minute": 0
            }
          ]
        }
      }
    },
    {
      "id": "ExecSSL",
      "name": "Check Expiry",
      "type": "n8n-nodes-base.executeCommand",
      "typeVersion": 1,
      "position": [600, 300],
      "parameters": {
        "command": "echo | openssl s_client -servername synlabs.com -connect synlabs.com:443 2>/dev/null | openssl x509 -noout -dates"
      }
    }
  ],
  "connections": {
    "Daily Check": {
      "main": [
        [
          {
            "node": "Check Expiry",
            "type": "main",
            "index": 0
          }
        ]
      ]
    }
  }
}



 


