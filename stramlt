import streamlit as st
import requests
import base64

st.set_page_config(page_title="GitHub Repo Creator", page_icon="🐙", layout="wide")

st.title("🐙 GitHub Repository Creator")
st.markdown("Create a new GitHub repository and upload your files directly from this app!")

# Sidebar for GitHub credentials
with st.sidebar:
    st.header("GitHub Credentials")
    github_token = st.text_input("GitHub Personal Access Token", type="password", help="Generate at: https://github.com/settings/tokens")
    st.markdown("[How to create a token](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token)")

# Main form
if github_token:
    # Verify token
    headers = {
        "Authorization": f"token {github_token}",
        "Accept": "application/vnd.github.v3+json"
    }
    
    user_response = requests.get("https://api.github.com/user", headers=headers)
    
    if user_response.status_code == 200:
        user_data = user_response.json()
        st.success(f"✅ Authenticated as: **{user_data['login']}**")
        
        # Repository details
        st.header("Repository Details")
        col1, col2 = st.columns(2)
        
        with col1:
            repo_name = st.text_input("Repository Name*", placeholder="my-awesome-project")
            description = st.text_area("Description", placeholder="A brief description of your project")
            is_private = st.checkbox("Private Repository", value=False)
        
        with col2:
            auto_init = st.checkbox("Initialize with README", value=True)
            gitignore_template = st.selectbox("Add .gitignore", ["None", "Python", "Node", "Java", "C++", "Go"])
            license_template = st.selectbox("Choose License", ["None", "MIT", "Apache-2.0", "GPL-3.0", "BSD-3-Clause"])
        
        # File upload section
        st.header("Upload Files")
        uploaded_files = st.file_uploader("Select files to upload", accept_multiple_files=True)
        
        if uploaded_files:
            st.write(f"**{len(uploaded_files)} file(s) selected:**")
            for file in uploaded_files:
                st.write(f"- {file.name}")
        
        # Create repository button
        if st.button("🚀 Create Repository", type="primary", use_container_width=True):
            if not repo_name:
                st.error("❌ Repository name is required!")
            else:
                with st.spinner("Creating repository..."):
                    # Prepare repository data
                    repo_data = {
                        "name": repo_name,
                        "description": description,
                        "private": is_private,
                        "auto_init": auto_init
                    }
                    
                    if gitignore_template != "None":
                        repo_data["gitignore_template"] = gitignore_template
                    
                    if license_template != "None":
                        repo_data["license_template"] = license_template
                    
                    # Create repository
                    create_response = requests.post(
                        "https://api.github.com/user/repos",
                        headers=headers,
                        json=repo_data
                    )
                    
                    if create_response.status_code == 201:
                        repo_info = create_response.json()
                        st.success(f"✅ Repository created successfully!")
                        st.markdown(f"**Repository URL:** [{repo_info['html_url']}]({repo_info['html_url']})")
                        
                        # Upload files if any
                        if uploaded_files:
                            st.info("📤 Uploading files...")
                            success_count = 0
                            
                            for file in uploaded_files:
                                file_content = file.read()
                                encoded_content = base64.b64encode(file_content).decode()
                                
                                file_data = {
                                    "message": f"Add {file.name}",
                                    "content": encoded_content
                                }
                                
                                upload_response = requests.put(
                                    f"https://api.github.com/repos/{user_data['login']}/{repo_name}/contents/{file.name}",
                                    headers=headers,
                                    json=file_data
                                )
                                
                                if upload_response.status_code == 201:
                                    success_count += 1
                                    st.success(f"✅ Uploaded: {file.name}")
                                else:
                                    st.error(f"❌ Failed to upload {file.name}: {upload_response.json().get('message', 'Unknown error')}")
                            
                            st.success(f"🎉 {success_count}/{len(uploaded_files)} file(s) uploaded successfully!")
                        
                        st.balloons()
                    
                    elif create_response.status_code == 422:
                        st.error("❌ Repository already exists or invalid name!")
                    else:
                        error_message = create_response.json().get('message', 'Unknown error')
                        st.error(f"❌ Failed to create repository: {error_message}")
    
    else:
        st.error("❌ Invalid GitHub token. Please check your credentials.")
        st.info("💡 Make sure your token has 'repo' scope enabled.")

else:
    st.info("👈 Please enter your GitHub Personal Access Token in the sidebar to get started.")
    
    with st.expander("ℹ️ How to get a GitHub Token"):
        st.markdown("""
        1. Go to [GitHub Settings > Tokens](https://github.com/settings/tokens)
        2. Click **Generate new token** (classic)
        3. Give it a name and select scopes:
           - ✅ `repo` (Full control of private repositories)
        4. Click **Generate token**
        5. Copy the token and paste it in the sidebar
        
        ⚠️ **Important:** Keep your token secure and never share it!
        """)

# Footer
st.markdown("---")
st.markdown("Made with ❤️ using Streamlit | [GitHub API Documentation](https://docs.github.com/en/rest)")
