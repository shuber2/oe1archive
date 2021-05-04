#!/usr/bin/env python3

"""A simple tool to query the Oe1 7 Tage archive."""

__version__ = "2.1"
__author__ = "Stefan Huber"


import urllib.request
import simplejson
import dateutil.parser
import sys
import getopt
import re
import os


class Archive:

    def __init__(self):
        self.json = read_json("http://audioapi.orf.at/oe1/json/2.0/broadcasts/")

    def get_days(self):
        return map(_json_to_day, self.json)

    def get_broadcasts(self, day):
        bjson = self.json[day]['broadcasts']
        return map(_json_to_broadcast, bjson)

    def get_broadcast(self, day, broadcast):
        return _json_to_broadcast(self.json[day]['broadcasts'][broadcast])

    def get_player_url(self, day, broadcast):
        date = self.json[day]['day']
        pk = self.json[day]['broadcasts'][broadcast]['programKey']
        url = "http://oe1.orf.at/player/%d/%s"
        return url % (date, pk)

    def get_broadcast_subtitle(self, day, broadcast):
        return self.json[day]['broadcasts'][broadcast]['subtitle']

    def get_broadcast_pk(self, day, broadcast):
        return self.json[day]['broadcasts'][broadcast]['programKey']

    def get_broadcast_url(self, day, broadcast):
        date = self.json[day]['day']
        pk = self.json[day]['broadcasts'][broadcast]['programKey']

        burl = 'https://audioapi.orf.at/oe1/api/json/current/broadcast/%s/%d'
        bjson = read_json(burl % (pk, date))

        sjson = bjson['streams']
        if len(sjson) == 0:
            return None

        sid = sjson[0]['loopStreamId']
        surl = 'https://loopstream01.apa.at/?channel=oe1&shoutcast=0&id=%s'
        return surl % sid

    def get_broadcast_description(self, day, broadcast):
        date = self.json[day]['day']
        pk = self.json[day]['broadcasts'][broadcast]['programKey']

        burl = 'https://audioapi.orf.at/oe1/api/json/current/broadcast/%s/%d'
        bjson = read_json(burl % (pk, date))

        description = bjson['description']
        akm = bjson['akm']
        if description is None:
            description = ""
        if akm is None:
            akm = ""
        return description + "<br>" + akm;

    def get_broadcasts_by_regex(self, key):
        rex = re.compile(key, re.IGNORECASE)

        res = []
        for d, djson in enumerate(self.json):
            for b, bjson in enumerate(djson['broadcasts']):
                if rex.search(bjson['title']) is not None:
                    res.append((d, b))
                elif bjson['subtitle'] is not None and rex.search(bjson['subtitle']) is not None:
                    res.append((d, b))
        return res

def _json_to_day(djson):
    return dateutil.parser.parse(djson['dateISO'])

def _json_to_broadcast(bjson):
    dt = dateutil.parser.parse(bjson['startISO'])
    return (dt, bjson['title'])


def read_json(url):
    with urllib.request.urlopen(url) as f:
        dec = simplejson.JSONDecoder()
        return dec.decode(f.read())

def input_index(prompt, li):
    while True:
        try:
            idx = int(input(prompt))
            if idx < 0 or idx >= len(li):
                print("Out out range!")
            else:
                return idx

        except ValueError:
            print("Unknown input.")
        except EOFError:
            sys.exit(1)

def screen_help():
    print("""Usage:
    {0} -h, --help
    {0} -c, --choose
    {0} -s, --search TITLE""".format(sys.argv[0]))

def screen_choose():
    a = Archive()

    print("Choose a date:")
    days = list(a.get_days())
    for i, date in enumerate(days):
        print("  [%d]  %s" % (i, date.strftime("%a %d. %b %Y")))
    day = input_index("Date: ", days)
    chosen_datetime = days[day]
    print()

    print("Choose a broadcast:")
    broadcasts = list(a.get_broadcasts(day))
    for i, b in enumerate(broadcasts):
        date, title = b
        print("  [%2d]  %s  %s" % (i, date.strftime("%H:%M:%S"), title))
    broadcast = input_index("Broadcast: ", broadcasts)
    print()

    print_broadcast_info(a, day, broadcast)
    print()

    url = a.get_broadcast_url(day, broadcast)
    if url is not None:
        answer = input("Do you want to download the chosen broadcast? (y/N) ")
        if answer in ["y", "Y", "j", "J"]:
            name = input("Download directory (prefix): ")

            try:
                dirname = get_directory_name(name, chosen_datetime)
                print("Downloading to %s..." % dirname)

                make_directory(name, chosen_datetime)

                description = a.get_broadcast_description(day, broadcast)
                write_html_file(name, chosen_datetime, description)

                write_mp3_file(name, chosen_datetime, url)

            except OSError as e:
                print("Error creating directory.")
                print(e)

            except requests.exceptions.RequestException as e:
                print("Request getting mp3 failed.")

            except Exception as e:
                print("Error downloading mp3.")
                print(e)

def get_directory_name(name, datetime):
    prefix = ""
    if len(name) > 0:
        prefix = name + "_"

    return prefix + datetime.strftime("%d-%m-%Y")

def make_directory(name, datetime):
    """Creates the download subdirectory for the given name and datetime."""
    dirname = get_directory_name(name, datetime)
    if not os.path.exists(dirname):
        os.makedirs(dirname)

def write_html_file(name, datetime, description):
    """Stores broadcast description into a html file."""

    longname = get_directory_name(name, datetime)
    filepath = os.path.join(longname, longname + ".html")
    file = open(filepath, 'w+')
    file.write("<!DOCTYPE html>\n")
    file.write("<html>\n")
    file.write("<head>\n")
    file.write("<title>\n")
    file.write("%s %s\n" % (name, datetime.strftime("%d.%m.%Y")))
    file.write("</title>\n")
    file.write("<meta charset = \"utf-8\">\n")
    file.write("</head>\n")
    file.write("<body>\n")
    file.write("%s %s" % (name, datetime.strftime("%d.%m.%Y")))
    file.write(description)
    file.write("</body>\n")
    file.write("</html>")
    file.close()

def write_mp3_file(name, datetime, url):
    import requests

    longname = get_directory_name(name, datetime)
    filepath = os.path.join(longname, longname + ".mp3")

    print("Fetching mp3...")
    r = requests.get(url, stream=True)
    if r.status_code == 200:
        with open(filepath, 'wb') as f:
            f.write(r.content)
    else:
        print("Error downloading mp3. Status code: %d" % r.status_code)

def screen_search(key):
    a = Archive()
    for d, b in a.get_broadcasts_by_regex(key):
        print_broadcast_info(a, d, b)
        print()

def print_broadcast_info(archive, day, broadcast):
    a, d, b = archive, day, broadcast
    date, title = a.get_broadcast(d, b)

    print("%s   %s" % (date.strftime("%a %d.%m.%Y  %H:%M:%S"), title))
    print("  %s" % a.get_broadcast_subtitle(d, b))
    print("  Broadcast: %s" % a.get_broadcast_url(d, b))
    print("  Player: %s" % a.get_player_url(d, b))
    print("  Program key: %s" % a.get_broadcast_pk(d, b))

if __name__ == "__main__":

    try:
        opts, args = getopt.getopt(sys.argv[1:], "hcs:",
                ["help", "choose", "search="])
    except getopt.GetoptError as err:
        print(err)
        screen_help()
        sys.exit(2)

    for o, a in opts:
        if o in ["-h", "--help"]:
            screen_help()
        if o in ["-c", "--choose"]:
            screen_choose()
        if o in ["-s", "--search"]:
            screen_search(a)
