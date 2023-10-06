STEPS.
**1.Bench Setup**
```
 git clone https://github.com/frappe/frappe_docker excel_bench
```
```
cd excel_bench
```
```
cp -R development/vscode-example development/.vscode
```
```
cp -R devcontainer-example .devcontainer
```
```
cp /path/to/20211024_150000-site_name_com-database.sql.gz development
```
```
wget https://gitlab.com/castlecraft/excel-rma/-/wikis/uploads/cda407fe7054c80f6ce4586d9f0f1982/docker-compose.yml -O .devcontainer/docker-compose.yml

```
```
code .
```
**Re-open in devcontainer and execute:**
```
# get apps.json
wget https://gitlab.com/castlecraft/excel-rma/-/wikis/uploads/489583f9bf1adc8258492c06253db61e/apps.json -O apps.json
```
```
# setup bench and excel.localhost
./installer.py -j apps.json -s excel.localhost -v
```
# restore site db
```
cd frappe-bench
```
```
bench --site excel.localhost restore --force /workspace/development/20211024_150000-site_name_com-database.sql.gz
```
**Post setup steps**
```
bench --site excel.localhost set-config allow_cors "\"*\""
bench --site excel.localhost set-admin-password admin
bench --site excel.localhost console
```
**In python console:**
```
a = frappe.get_doc("User", "bot@example.com")
a.api_key = "6b045a3f08fe3f0"
a.api_secret = "e981452172beb4b"
a.save()
frappe.db.commit()
```
```
for o in frappe.get_all("OAuth Client"):
   oc = frappe.get_doc("OAuth Client", o.get("name"))
   oc.redirect_uris = oc.redirect_uris.replace("https://testrma.example.com", "http://rma.localhost:4700")
   oc.default_redirect_uri = oc.default_redirect_uri.replace("https://testrma.example.com", "http://rma.localhost:4700")
   if oc.grant_type == "Implicit":
     oc.grant_type = "Authorization Code"
     oc.response_type = "Code"
   oc.save()
   frappe.db.commit()
```
```
for w in frappe.get_all("Webhook", filters={"webhook_doctype": ["not in", "Data Import Legacy"]}):
   webhook = frappe.get_doc("Webhook", w.get("name"))
   webhook.request_url = webhook.request_url.replace("https://testrma.example.com", "http://rma.localhost:4700")
   webhook.save()
   frappe.db.commit()

```
**Exit from Console**
```
exit
```
**Migrate Once**
```
bench --site excel.localhost migrate
```

**STEP 2**
**How to start RMA Server**
Go to main terminal:
```
git clone https://gitlab.com/castlecraft/excel-rma
```
```
code excel-rma
```
**
Copy here Dump file
**
```
cp -fR /path/to/dump restore/dump

```
**Restore MongoDB Dump**
```
mongorestore -u rma-server -p admin -h mongodb --authenticationDatabase=rma-server --db rma-server ./restore/dump/rma-server
```

**Reopen in devcontainer**
```
cd packages/rma-server
cp env-devcontainer .env
```
**Next, let's run our server.**
```
yarn
yarn start:debug
```
**See Your site here**
```
http://rma.localhost:4700/home
```

What about frontend(s)?
This app has 2 frontends

packages/rma-frontend
packages/rma-warranty

**How to start rma-frontend?**
```
cd packages/rma-frontend
yarn
yarn start
```
**How to start rma-warranty?**
```
cd packages/rma-warranty
yarn
yarn start
```












