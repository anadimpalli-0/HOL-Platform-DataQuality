# How to Complete Lab Grading

Congrats\! You have completed the lab. Please run the following commands in Snowsight to confirm your completion.

Remember to edit your contact information in the SQL Statement for the [SE_GREETER.sql](/config/SE_GREETER.sql)

---
## ⚠️ Important Notes

### Before Running Validation:
- **Replace `<YOUR_USERNAME>`** in the script with your actual Snowflake username
  ```sql
  SET username = '<YOUR_USERNAME>';  -- Change to your actual username (usually in Uppercase)
  ```
- Ensure you're using the **ACCOUNTADMIN** role when running validation
- Complete all phases **before cleanup** 
---


* [Greeter Script for DORA](/config/SE_GREETER.sql)
* [Grading Script for DORA](/config/DoraGrading.sql)

If all validations return ✅, you have successfully completed the HOL

---
# Next Steps
## 🧹 Cleanup

- Run these SQL commands to cleaup the objects created in this lab

```sql```

 USE ROLE ACCOUNTADMIN;

 DROP DATABASE dq_tutorial_db;

 DROP WAREHOUSE dq_tutorial_wh;

 DROP ROLE dq_tutorial_role;

---