## Redirect Checker

Here's a Python 3 script that takes a text file of domain names as input and checks if any of them redirect to a different URL:

```
import requests

def check_redirects(url):
    try:
        response = requests.get(url, allow_redirects=False)
        if response.status_code in (301, 302, 303, 307, 308):
            return response.headers['Location']
        return None
    except requests.RequestException:
        return None

def main():
    input_file = input("Enter the path to the input file: ")
    
    with open(input_file, 'r') as file:
        domains = [line.strip() for line in file]
    
    for domain in domains:
        url = f"http://{domain}"
        redirect_url = check_redirects(url)
        
        if redirect_url:
            print(f"{url} redirects to {redirect_url}")
        else:
            print(f"{url} does not redirect")

if __name__ == "__main__":
    main()
```
This script does the following:

1. We import the requests library to handle HTTP requests
2. The check_redirects function takes a URL as input and checks if it redirects:
3. It sends a GET request with allow_redirects=False to prevent automatic following of redirects
4. If the status code indicates a redirect (301, 302, 303, 307, or 308), it returns the redirect location
5. If there's no redirect or an error occurs, it returns None.
    
For the main function:
1. Prompts the user for the input file path.
2. Reads the domain names from the file, stripping any whitespace.
3. For each domain, it prepends "http://" to create a valid URL.
4. Calls check_redirects for each URL and prints the result.

To use this script:

1. Save it as a .py file (e.g., check_redirects.py).
2. Create a text file with one domain name per line (e.g., domains.txt).
3. Run the script and enter the path to your input file when prompted.

The script will then check each domain and report whether it redirects or not. Note that this script only checks for immediate redirects and doesn't follow redirect chains

***


## Modification 1:
Check for redirects regardless of if its HTTP or HTTPS:
```
import requests

def check_redirects(url):
    try:
        response = requests.get(url, allow_redirects=False, timeout=5)
        if response.status_code in (301, 302, 303, 307, 308):
            return response.headers['Location']
        return None
    except requests.RequestException:
        return None

def main():
    input_file = input("Enter the path to the input file: ")
    
    with open(input_file, 'r') as file:
        domains = [line.strip() for line in file]
    
    for domain in domains:
        for protocol in ['http', 'https']:
            url = f"{protocol}://{domain}"
            redirect_url = check_redirects(url)
            
            if redirect_url:
                print(f"{url} redirects to {redirect_url}")
            else:
                print(f"{url} does not redirect")

if __name__ == "__main__":
    main()
```
Here are the key modifications and improvements:

1. In the main() function, we now iterate over both 'http' and 'https' protocols for each domain.
2. The url is constructed using an f-string with the current protocol and domain.
3. We check for redirects for both HTTP and HTTPS versions of each domain.
4. Added a timeout parameter to the requests.get() call in the check_redirects() function to prevent hanging on slow or unresponsive servers.
5. The script will now print the results for both HTTP and HTTPS versions of each domain, showing whether they redirect or not.

This modified version will check for redirects on both HTTP and HTTPS protocols for each domain in the input file, giving you a more comprehensive view of the redirect behavior for each domain.

***

Modification 2

Modify the script to follow redirect chains for both HTTP and HTTPS. To do this you can use the requests library, which handles redirects more elegantly than urllib2. Here's an updated version of the script:

```
import requests

def check_redirects(url):
    try:
        response = requests.get(url, allow_redirects=True, timeout=5)
        redirect_history = [r.url for r in response.history]
        final_url = response.url
        return redirect_history, final_url
    except requests.RequestException as e:
        return None, str(e)

def main():
    input_file = input("Enter the path to the input file: ")
    
    with open(input_file, 'r') as file:
        domains = [line.strip() for line in file]
    
    for domain in domains:
        for protocol in ['http', 'https']:
            url = f"{protocol}://{domain}"
            redirect_history, final_url = check_redirects(url)
            
            if redirect_history:
                print(f"{url} redirects:")
                for i, redirect in enumerate(redirect_history, 1):
                    print(f"  {i}. {redirect}")
                print(f"Final URL: {final_url}")
            else:
                print(f"{url} does not redirect. Error: {final_url}")
            print()  # Add a blank line for readability

if __name__ == "__main__":
    main()
```
This modified script offers several improvements:

1. It uses the requests library, which automatically handles redirect chains
2. The check_redirects function now returns both the redirect history and the final URL
3. It attempts both HTTP and HTTPS connections for each domain
4. The script prints out the entire redirect chain, if any, along with the final URL
5. It includes error handling to catch and report any issues during the request
6. A timeout of 5 seconds is set to prevent hanging on slow or unresponsive servers
7. The output is formatted to clearly show the redirect chain and final URL for each domain and protocol combination.

This script will follow all redirects in the chain, regardless of whether they are HTTP or HTTPS, and will display the complete path of redirections for each URL. It provides a comprehensive view of the redirect behavior for each domain in your input file.


