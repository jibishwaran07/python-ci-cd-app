stage('Cleanup Docker Hub Tags (Keep Last 3)') {
    steps {
        withCredentials([usernamePassword(
            credentialsId: 'dockerhub-creds',
            usernameVariable: 'DH_USER',
            passwordVariable: 'DH_PASS'
        )]) {
            sh '''
            set -e

            echo "Getting Docker Hub JWT token"
            TOKEN=$(curl -s -X POST https://hub.docker.com/v2/users/login/ \
              -H "Content-Type: application/json" \
              -d "{\"username\":\"$DH_USER\",\"password\":\"$DH_PASS\"}" \
              | jq -r .token)

            echo "JWT Token obtained: $TOKEN"

            echo "Fetching tags from Docker Hub"
            TAGS=$(curl -s -H "Authorization: JWT $TOKEN" \
              https://hub.docker.com/v2/repositories/'"$REPO"'/tags?page_size=100 \
              | jq -r '.results | sort_by(.last_updated) | reverse | .[].name')

            count=0
            for tag in $TAGS; do
              count=$((count + 1))
              if [ $count -gt 3 ]; then
                echo "Deleting old tag: $tag"
                curl -s -X DELETE \
                  -H "Authorization: JWT $TOKEN" \
                  https://hub.docker.com/v2/repositories/'"$REPO"'/tags/$tag/
              fi
            done
            '''
        }
    }
}
