# davidwebb-scrapping-
Website scrapping (create pdf)
# install requirement.txt
# copy code and run it

import csv
import datetime
import os
import requests
from bs4 import BeautifulSoup
import logging
from reportlab.lib import colors
from reportlab.lib.pagesizes import letter
from reportlab.lib.enums import TA_JUSTIFY, TA_CENTER
from reportlab.platypus import SimpleDocTemplate, Paragraph, Spacer, Image, Table, TableStyle
from reportlab.lib.styles import getSampleStyleSheet, ParagraphStyle
from reportlab.lib.units import inch


logging.basicConfig(filename='web_site.log', format='%(asctime)s: %(process)d-%(levelname)s-%(message)s', level=logging.DEBUG)

elements = []
date_time = datetime.datetime.now()
date = date_time.date()
path = '/Users/Alice/Downloads/alice/'+str(date)

def get_tables(code, path):
    code = str(code)
    print('\033[1m \33[95m [+] \33[95m Requesting for '+code+' \033[0m')
    base_url = 'https://webb-site.com'
    urls = 'https://webb-site.com/dbpub/orgdata.asp?code='+code+'&Submit=current'
    agent = ['Mozilla/5.0 (Windows NT 6.1; WOW64; rv:54.0) Gecko/20100101 Firefox/72.0','Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/79.0.3945.130 Safari/537.36']
    headers = {'user-agent': agent[0]}
    page = requests.get(urls, headers=headers)

    def bs_parse(content):
        return BeautifulSoup(content, 'html.parser')

    soup = bs_parse(page.content)

    ccass = ''
    for link in soup.find_all('a'):
        href = link.get('href')
        if type(href) is str:
            if 'ccass' in href:
                ccass = href
                break

    print('\033[1m \33[92m [+] \33[92m Now getting CCASS for '+code+' \033[0m')
    ccass_page = requests.get(base_url+ccass, headers=headers)
    ccass_soup = bs_parse(ccass_page.content)



    # get holding...

    holding_table_data = []
    holdings_table = ccass_soup.find_all('table', class_='optable')
    title = ccass_soup.find('h3').text.strip()
    table1 = holdings_table[0]
    table1_tr = table1.find_all('tr')

    all_th = []
    for th in table1.find_all('th'):
        if th.text.strip() != '':
            all_th.append(th.text.strip())

    holding_table_data.append(all_th)

    all_row = []
    for td in table1_tr:
        all_td = td.find_all('td')
        if len(all_td) > 0:
            holding_table_data.append([all_td[0].text, all_td[1].text, all_td[2].text])


    # print('++++++++++++++++++++++++++++++++')
    # print(holding_table_data)
    # print('++++++++++++++++++++++++++++++++')

    sp = ParagraphStyle('parrafos',
                            alignment=TA_CENTER,
                            fontSize=14,
                            fontName="Times-Roman",)
    ptext = '<font size=17 color="red">'+code+' - '+title+'</font>'
    ptext3 = '<font size=12 color="red">Summary</font>'

    data = holding_table_data
    t=Table(data)
    t.setStyle(TableStyle([
                       ('BOX', (0,0), (-1,-1), 0.25, colors.black),
                       ('INNERGRID', (0,0), (-1,-1), 0.25, colors.black),
                       ]))
    elements.append(Spacer(7, 12))
    elements.append(Paragraph(ptext, sp))
    elements.append(Spacer(2, 3))
    elements.append(Paragraph(ptext3, sp))
    elements.append(Spacer(7, 12))
    elements.append(t)


    # Details......

    table2 = holdings_table[1]
    table2_tr = table2.find_all('tr')

    details_table_data = []
    all_th = []
    for th in table2.find_all('th'):
        all_th.append(th.text)

    details_table_data.append(all_th)

    for td in table2_tr:
        all_td = td.find_all('td')
        if len(all_td) > 0:
            if int(all_td[0].text.strip()) < 11:
                details_table_data.append([all_td[0].text.strip(), all_td[1].text, all_td[2].text, all_td[3].text, all_td[4].text, all_td[5].text.strip(), all_td[6].text])
            else:
                break

    sp = ParagraphStyle('parrafos',
                            alignment=TA_CENTER,
                            fontSize=14,
                            fontName="Times-Roman",)
    ptext = '<font size=17 color="red">'+code+' - Details</font>'
    data = details_table_data
    t=Table(data)
    t.setStyle(TableStyle([
                       ('BOX', (0,0), (-1,-1), 0.25, colors.black),
                       ('INNERGRID', (0,0), (-1,-1), 0.25, colors.black),
                       ('FONTSIZE', (0, 0), (-1, -1), 10)
                       ]))
    elements.append(Spacer(7, 12))
    elements.append(Paragraph(ptext, sp))
    elements.append(Spacer(7, 12))
    elements.append(t)



    changes = ''
    concentration = ''

    for link in ccass_soup.find_all('a'):
        href = link.get('href')
        # print(href)
        if type(href) is str:
            if 'chldchg' in href:
                changes = href

            if 'cconchist' in href:
                concentration = href

            if concentration != '' and changes != '':
                break


    print('\033[1m \33[92m [+] \33[92m Now getting Changes for '+code+' \033[0m')
    url = base_url+'/ccass/'+ changes


    changes_page = requests.get(url, headers=headers)
    changes_soup = bs_parse(changes_page.content)

    changes_table = changes_soup.find('table', class_='optable')
    title = changes_soup.find('h3').text.strip()

    table_tr = changes_table.find_all('tr')

    change_table_data = []
    all_th = []
    for th in changes_table.find_all('th'):
        all_th.append(th.text)
    change_table_data.append(all_th)

    for td in table_tr:
        all_td = td.find_all('td')

        if len(all_td) > 0:
            change_table_data.append([all_td[0].text.strip(), all_td[1].text, all_td[2].text, all_td[3].text, all_td[4].text, all_td[5].text, all_td[6].text.strip(), all_td[7].text])


    sp = ParagraphStyle('parrafos',
                            alignment=TA_CENTER,
                            fontSize=14,
                            fontName="Times-Roman",)
    ptext = '<font size=17 color="red">'+code+' - '+title+'</font>'

    data = change_table_data
    t=Table(data)
    t.setStyle(TableStyle([
                       ('BOX', (0,0), (-1,-1), 0.25, colors.black),
                       ('INNERGRID', (0,0), (-1,-1), 0.25, colors.black),
                       ]))
    elements.append(Spacer(7, 12))
    elements.append(Paragraph(ptext, sp))
    elements.append(Spacer(4, 5))
    elements.append(Paragraph(ptext3, sp))
    elements.append(Spacer(7, 12))
    elements.append(t)

    print('\033[1m \33[92m [+] \33[92m Now getting Concentration for '+code+' \033[0m')
    url = base_url+'/ccass/'+concentration
    concentration_page = requests.get(url, headers=headers)
    concentration_soup = bs_parse(concentration_page.content)


    concentration_table = concentration_soup.find_all('table', class_='numtable')
    table_tr = concentration_table[1].find_all('tr')

    all_th = []
    concentration_table_data = []

    for th in concentration_table[1].find_all('th'):
        all_th.append(th.text)

    concentration_table_data.append(all_th)

    for td in table_tr:
        all_td = td.find_all('td')

        if len(all_td) > 0:
            if int(all_td[0].text.strip()) < 21:
                concentration_table_data.append([all_td[0].text.strip(), all_td[1].text, all_td[2].text, all_td[3].text, all_td[4].text, all_td[5].text])


    sp = ParagraphStyle('parrafos',
                            alignment=TA_CENTER,
                            fontSize=14,
                            fontName="Times-Roman",)
    ptext = '<font size=17 color="red">'+code+' - CCASS concentration analysis</font>'

    data = concentration_table_data
    t=Table(data)
    t.setStyle(TableStyle([
                       ('BOX', (0,0), (-1,-1), 0.25, colors.black),
                       ('INNERGRID', (0,0), (-1,-1), 0.25, colors.black),
                       ]))
    elements.append(Spacer(7, 12))
    elements.append(Paragraph(ptext, sp))
    elements.append(Spacer(7, 12))
    elements.append(t)

    print('\033[1m \33[92m [+] \33[92m Done for '+code+' \033[0m')
    print()

def mkdir():
    try:
        os.makedirs(path)
    except OSError as e:
        pass

        #logging.error('Creation of the directory '+path+'failed. '+e)
    else:
        logging.info('Successfully created the directory '+path)


def main():
    mkdir()
    pdf_path = path+"/"+str(date)+".pdf"
    doc = SimpleDocTemplate(pdf_path, pagesize=letter)
    code_list = [859, 948, 1321, 1367, 1536, 8187, 1753]
    # code_list = [859]

    for code in code_list:
        get_tables(code, path)
    print(len(elements))
    doc.build(elements)
    print('\033[1m \33[93m [+] \33[93m Pdf file is stored here "'+path+'" \033[0m')

main()
