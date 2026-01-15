```sh
# 1. Download js files from the website say ~/target-js
> wget --recursive --no-clobber --page-requisites --html-extension --convert-links --restrict-file-names=windows --domains example.com --no-parent https://example.com/ # Mirror the website (this downloads HTML, JS, CSS, etc., to a local directory)
> mkdir ~/target-js 
> find example.com/ -type f -name "*.js" -exec cp {} ~/target-js/ \;
> cd ~/target-js/ 
> for file in *.min.js; do mv "$$ file" " $${file%.min.js}.js"; done
> cd ~/target-js/ 
> mkdir beautified 
> for file in *.js; do js-beautify "$file" > "beautified/$file"; done
> cd ~/target-js/beautified/

# 2. Create CodeQL database.
codeql database create ~/target-db --language=javascript --source-root=~/target-js/beautified/

# 3. Run queries
codeql database analyze ~/target-db javascript-security-extended.qls --format=sarif-latest --output=results.sarif
# Target specific query
codeql database analyze ~/target-db codeql/javascript-queries/Security/CWE-079/XssThroughDom.ql --format=text --output=results.txt
```

```sh
# Code Scanning suite: Queries run by default in CodeQL code scanning on GitHub.  
codeql database analyze codeqldb $CODEQL_SUITES_PATH/javascript-code-scanning.qls \  
--format=sarifv2.1.0 \  
--output=$RESULTS_FOLDER/javascript-code-scanning.sarif  
  
# Security extended suite: Queries of lower severity and precision than the default queries  
codeql database analyze codeqldb $CODEQL_SUITES_PATH/javascript-security-extended.qls \  
--format=sarif-latest \  
--output=$RESULTS_FOLDER/javascript-security-extended.sarif  
  
# Security and quality suite: Queries of lower severity and precision than the default queries  
codeql database analyze codeqldb $CODEQL_SUITES_PATH/javascript-security-and-quality.qls \  
--format=sarif-latest \  
--output=$RESULTS_FOLDER/javascript-security-and-quality.sarif
```