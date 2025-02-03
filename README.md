Django istifadə edərək bir saytdan məlumatları çəkib (web scraping), bazaya əlavə etmək üçün aşağıdakı addımları izləyə bilərsiniz:

1. Lazımi Kitabxanaları Quraşdırın

Terminalda aşağıdakı komandaları işlədin:

pip install requests beautifulsoup4

	•	requests – Saytdan HTML məlumatlarını çəkmək üçün
	•	beautifulsoup4 – HTML-dən məlumatları çıxarmaq üçün

2. Django Modelinizi Yaradın

Məlumatları bazaya saxlamaq üçün models.py faylınıza uyğun modeli əlavə edin:

from django.db import models

class Article(models.Model):
    title = models.CharField(max_length=255)
    content = models.TextField()
    url = models.URLField(unique=True)
    created_at = models.DateTimeField(auto_now_add=True)

    def __str__(self):
        return self.title

Daha sonra modeli tətbiq etmək üçün aşağıdakı komandaları işlədin:

python manage.py makemigrations
python manage.py migrate

3. Scraper Skriptini Yaradın

Yeni bir fayl yaradın, məsələn, scraper.py və kodu əlavə edin:

import requests
from bs4 import BeautifulSoup
from yourapp.models import Article  # Django modelinizi əlavə edin

def scrape_website():
    url = "https://example.com"  # Buraya məlumatları çəkmək istədiyiniz saytı yazın
    response = requests.get(url)
    
    if response.status_code == 200:
        soup = BeautifulSoup(response.text, 'html.parser')
        
        # Saytın strukturuna uyğun başlıq və məzmunu çıxarın
        articles = soup.find_all("div", class_="article")  # HTML strukturunu yoxlayın
        
        for article in articles:
            title = article.find("h2").text.strip()
            content = article.find("p").text.strip()
            article_url = article.find("a")["href"]
            
            # Məlumatı bazaya əlavə et (əgər mövcud deyilsə)
            obj, created = Article.objects.get_or_create(
                url=article_url,
                defaults={"title": title, "content": content}
            )

            if created:
                print(f"Added: {title}")
            else:
                print(f"Already exists: {title}")

    else:
        print(f"Error: {response.status_code}")

4. Scraper-i Django Management Command Kimi Yaradın (Opsional)

Əgər scraper-i Django komanda kimi işlətmək istəyirsinizsə:

Komanda üçün fayl yaradın:

yourapp/management/commands/scrape.py

Kod əlavə edin:

from django.core.management.base import BaseCommand
from yourapp.scraper import scrape_website  # Scraper funksiyanı import edin

class Command(BaseCommand):
    help = "Scrapes data from a website and saves to database"

    def handle(self, *args, **kwargs):
        scrape_website()

Komandanı işlədin:

python manage.py scrape

5. Scraper-i Periodik İşlətmək (Opsional)

Əgər scraping prosesini mütəmadi işlətmək istəyirsinizsə:
	•	Celery və ya cronjob istifadə edə bilərsiniz.

Məsələn, cronjob üçün crontab -e açıb aşağıdakı sətri əlavə edin:

0 * * * * /path/to/your/venv/bin/python /path/to/your/manage.py scrape

Bu, hər saatda bir dəfə scraper-i işlədəcək.

Sonuç

İndi Django layihəniz üçün bir scraper hazırladınız. Bu sistem saytdan məlumatları çəkəcək və bazaya əlavə edəcək. Əlavə olaraq, Django Management Command və Cronjob ilə avtomatik işlədə bilərsiniz.

Saytın HTML strukturuna uyğun olaraq BeautifulSoup kodunu tənzimləməyi unutmayın.
