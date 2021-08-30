# cloudflare

### 准备数据
export CF_API_EMAIL=xxxxxx@gmail.com

export CF_API_KEY=71f4972b518c4a
### 新增域名
curl -X POST -H "X-Auth-Key: $CF_API_KEY" -H "X-Auth-Email: $CF_API_EMAIL" -H "Content-Type: application/json" "https://api.cloudflare.com/client/v4/zones" \
--data '{"name":"'abc.com'","jump_start":true}'
### 删除域名
curl -X DELETE "https://api.cloudflare.com/client/v4/zones/aefe761d4e5fd7f2008a92c7755cb70e" \
     -H "X-Auth-Key: $CF_API_KEY" -H "X-Auth-Email: $CF_API_EMAIL" -H "Content-Type: application/json"
### 增加域名解析
curl -X POST "https://api.cloudflare.com/client/v4/zones/8a3e9971b8605dee22c25b79f33530f4/dns_records" \
     -H "X-Auth-Key: $CF_API_KEY" -H "X-Auth-Email: $CF_API_EMAIL" -H "Content-Type: application/json" \
curl -X POST "https://api.cloudflare.com/client/v4/zones/181212784295db0ffd9071a47a7d5cbd/dns_records" \
     -H "X-Auth-Key: $CF_API_KEY" -H "X-Auth-Email: $CF_API_EMAIL" -H "Content-Type: application/json" \
     --data '{"type":"A","name":"www.abc.com","content":"194.0.0.1","proxied":true}'

### 批量前准备数据
域名列表 domain.txt
curl -X GET "https://api.cloudflare.com/client/v4/zones?page=1&per_page=500&order=status&direction=desc&match=all" \
    -H "X-Auth-Key: $CF_API_KEY" -H "X-Auth-Email: $CF_API_EMAIL" -H "Content-Type: application/json" |jq  '.result[] | [.id,.name] | join(":")' >>zone.txt
sed -i 's/"//g' zone.txt
### 批量删除域名
for domain in $(cat domain.txt); do \
	curl -X DELETE "https://api.cloudflare.com/client/v4/zones/$(cat zone.txt|grep $domain |awk -F ':' '{print $1}')" \
     -H "X-Auth-Key: $CF_API_KEY" -H "X-Auth-Email: $CF_API_EMAIL" -H "Content-Type: application/json"; done
### 批量新增域名
for domain in $(cat domain.txt); do curl -X POST -H "X-Auth-Key: $CF_API_KEY" -H "X-Auth-Email: $CF_API_EMAIL" -H "Content-Type: application/json" "https://api.cloudflare.com/client/v4/zones" \
--data '{"name":"'$domain'","jump_start":true}'; done
### 批量增加域名解析
for domain in $(cat domain.txt); do \
curl -X POST "https://api.cloudflare.com/client/v4/zones/$(cat zone.txt|grep $domain |awk -F ':' '{print $1}')/dns_records" \
     -H "X-Auth-Key: $CF_API_KEY" -H "X-Auth-Email: $CF_API_EMAIL" -H "Content-Type: application/json" \
     --data '{"type":"cname","name":"www.'$domain'","content":"abc.com","ttl":120,"priority":10,"proxied":true}' \ ; done

A记录

for domain in $(cat domain.txt); do \
curl -X POST "https://api.cloudflare.com/client/v4/zones/$(cat zone.txt|grep $domain |awk -F ':' '{print $1}')/dns_records" \
     -H "X-Auth-Key: $CF_API_KEY" -H "X-Auth-Email: $CF_API_EMAIL" -H "Content-Type: application/json" \
     --data '{"type":"A","name":"'$domain'","content":"194.0.0.1","ttl":120,"priority":10,"proxied":true}' \ ; done
