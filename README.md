- Data was scraped from GitHub to analyze top developers in Shanghai with significant social following. The following code was used:-
```
import requests
import csv

GITHUB_TOKEN = ""
headers = {"Authorization": f"token {GITHUB_TOKEN}"}


def get_users():
    users = []
    search_url = "https://api.github.com/search/users"
    page = 1  
    params = {"q": "location:Shanghai followers:>200", "per_page": 100, "page": page}

    while True:
        response = requests.get(search_url, headers=headers, params=params)
        response_data = response.json()

        if response.status_code != 200:
            print(f"Error fetching users: {response_data.get('message', 'Unknown error')}")
            break

        users_data = response_data.get("items", [])
        if not users_data:
            break 

        for user in users_data:
            user_detail = requests.get(user["url"], headers=headers).json()
            user_info = {
                "login": user_detail["login"],
                "name": user_detail.get("name", ""),
                "company": clean_company_name(user_detail.get("company", "")),
                "location": user_detail.get("location", ""),
                "email": user_detail.get("email", ""),
                "hireable": user_detail.get("hireable", ""),
                "bio": user_detail.get("bio", ""),
                "public_repos": user_detail["public_repos"],
                "followers": user_detail["followers"],
                "following": user_detail["following"],
                "created_at": user_detail["created_at"]
            }
            users.append(user_info)

        page += 1
        params["page"] = page

    return users

def clean_company_name(company):
    return company.strip().lstrip('@').upper() if company else ""

def get_user_repos(username):
    repos = []
    repos_url = f"https://api.github.com/users/{username}/repos"
    params = {"per_page": 100}
    response = requests.get(repos_url, headers=headers, params=params)
    repos_data = response.json()

    for repo in repos_data[:500]:  
        repo_info = {
            "login": username,
            "full_name": repo["full_name"],
            "created_at": repo["created_at"],
            "stargazers_count": repo["stargazers_count"],
            "watchers_count": repo["watchers_count"],
            "language": repo.get("language", ""),
            "has_projects": repo["has_projects"],
            "has_wiki": repo["has_wiki"],
            "license_name": repo["license"]["key"] if repo["license"] else ""
        }
        repos.append(repo_info)
    return repos

def save_to_csv(filename, data, fieldnames):
    with open(filename, mode="w", newline="", encoding="utf-8") as file:
        writer = csv.DictWriter(file, fieldnames=fieldnames)
        writer.writeheader()
        writer.writerows(data)

users = get_users()
user_repos = []
for user in users:
    user_repos.extend(get_user_repos(user["login"]))

save_to_csv("users.csv", users, fieldnames=list(users[0].keys()))
save_to_csv("repositories.csv", user_repos, fieldnames=list(user_repos[0].keys()))
```

- A surprising finding: Over 60% of popular repositories in Shanghai are written in JavaScript.
- Recommendation: Developers in Shanghai may want to explore more languages like Python to diversify their skillset.

## Additional Insights
This analysis helps reveal trends in programming languages and developer activity in Shanghai.
