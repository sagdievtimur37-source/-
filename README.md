from django.db import models
 
from .utils import next_due_date
 
 
class Department(models.Model):
    name = models.CharField("Подразделение", max_length=200, unique=True)
 
    def __str__(self):
        return self.name
 
 
class Employee(models.Model):
    full_name = models.CharField("ФИО", max_length=200)
    position = models.CharField("Должность", max_length=200)
    department = models.ForeignKey(
        Department, on_delete=models.PROTECT, related_name="employees"
    )
    hired_at = models.DateField("Дата приема")
 
    def __str__(self):
        return self.full_name
 
 
class Examination(models.Model):
    class Kind(models.TextChoices):
        PRELIMINARY = "pre", "Предварительный"
        PERIODIC = "per", "Периодический"
 
    class Conclusion(models.TextChoices):
        FIT = "fit", "Годен"
        LIMITED = "lim", "Годен с ограничениями"
        UNFIT = "unf", "Не годен"
 
    employee = models.ForeignKey(
        Employee, on_delete=models.CASCADE, related_name="examinations"
    )
    kind = models.CharField(max_length=3, choices=Kind.choices)
    passed_at = models.DateField("Дата осмотра")
    conclusion = models.CharField(max_length=3, choices=Conclusion.choices)
    period_months = models.PositiveSmallIntegerField("Периодичность, мес.", default=12)
    next_due = models.DateField("Следующий осмотр", editable=False)
 
    class Meta:
        ordering = ["-passed_at"]
 
    def save(self, *args, **kwargs):
        # Диагнозы не хранятся: только дата и итоговое заключение.
        self.next_due = next_due_date(self.passed_at, self.period_months)
        super().save(*args, **kwargs)
 
    def __str__(self):
        return f"{self.employee} — {self.passed_at:%d.%m.%Y}"
