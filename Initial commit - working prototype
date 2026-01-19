import json
import os
from enum import Enum
from typing import Dict, Any, List, Optional
from dataclasses import dataclass, asdict
from datetime import datetime
import sys


class UserRole(Enum):
    JUDGE = "судья"
    LAWYER = "адвокат"
    PROSECUTOR = "прокурор"


@dataclass
class LegalCase:
    case_id: str
    description: str
    case_type: str
    parties: List[str]
    claims: List[str]
    documents: List[str]
    created_at: str
    status: str = "в работе"


@dataclass
class AnalysisResult:
    similar_cases: List[Dict]
    legal_norms: List[str]
    risk_score: float
    recommendations: List[str]
    contradictions: List[str]
    predicted_outcome: str = ""


@dataclass
class PersonalizedReport:
    role: UserRole
    generated_at: str
    case_id: str
    key_insights: List[str]
    actions: List[str]
    warnings: List[str]
    raw_analysis: AnalysisResult


class DataManager:
    def __init__(self, data_dir: str = "data"):
        self.data_dir = data_dir
        self.cases_file = os.path.join(data_dir, "cases.json")
        self.reports_dir = os.path.join(data_dir, "reports")
        self._ensure_directories()

    def _ensure_directories(self):
        os.makedirs(self.data_dir, exist_ok=True)
        os.makedirs(self.reports_dir, exist_ok=True)
        if not os.path.exists(self.cases_file):
            with open(self.cases_file, 'w', encoding='utf-8') as f:
                json.dump([], f)

    def save_case(self, case: LegalCase) -> bool:
        try:
            with open(self.cases_file, 'r', encoding='utf-8') as f:
                cases = json.load(f)
            case_dict = asdict(case)
            cases.append(case_dict)
            with open(self.cases_file, 'w', encoding='utf-8') as f:
                json.dump(cases, f, ensure_ascii=False, indent=2)
            return True
        except Exception as e:
            print(f"Ошибка при сохранении дела: {e}")
            return False

    def load_cases(self) -> List[Dict]:
        try:
            with open(self.cases_file, 'r', encoding='utf-8') as f:
                return json.load(f)
        except Exception as e:
            print(f"Ошибка при загрузке дел: {e}")
            return []

    def save_report(self, report: PersonalizedReport):
        try:
            report_dict = asdict(report)
            report_dict['role'] = report_dict['role']['value']
            filename = f"report_{report.case_id}_{report.role.value}_{datetime.now().strftime('%Y%m%d_%H%M%S')}.json"
            filepath = os.path.join(self.reports_dir, filename)
            with open(filepath, 'w', encoding='utf-8') as f:
                json.dump(report_dict, f, ensure_ascii=False, indent=2)
            return filepath
        except Exception as e:
            print(f"Ошибка при сохранении отчета: {e}")
            return None


class LegalAnalyticsCore:
    @staticmethod
    def analyze_case(case: LegalCase) -> AnalysisResult:
        case_type = case.case_type.lower()
        if "арбитраж" in case_type or "граждан" in case_type:
            similar_cases = [
                {"id": "А40-123456/2023", "court": "АС г. Москвы", "outcome": "иск удовлетворен", "similarity": 0.87},
                {"id": "А41-789012/2022", "court": "АС Московской области", "outcome": "частично удовлетворен",
                 "similarity": 0.76},
                {"id": "А43-456789/2023", "court": "АС г. Санкт-Петербурга", "outcome": "иск удовлетворен",
                 "similarity": 0.68}
            ]
            legal_norms = ["ст. 15 ГК РФ", "ст. 395 ГК РФ", "п. 3 ст. 10 ГК РФ", "ст. 309 ГК РФ"]
            predicted_outcome = "частичное удовлетворение"
        elif "уголов" in case_type:
            similar_cases = [
                {"id": "1-123/2023", "court": "Мосгорсуд", "outcome": "обвинительный приговор", "similarity": 0.92},
                {"id": "2-456/2022", "court": "Санкт-Петербургский горсуд", "outcome": "оправдательный приговор",
                 "similarity": 0.45}
            ]
            legal_norms = ["ст. 105 УК РФ", "ст. 158 УК РФ", "ст. 30 УК РФ"]
            predicted_outcome = "обвинительный приговор"
        else:
            similar_cases = [
                {"id": "33а-1234/2023", "court": "Верховный суд РФ", "outcome": "отказ в удовлетворении",
                 "similarity": 0.55}
            ]
            legal_norms = ["ст. 46 Конституции РФ", "ст. 125 ГПК РФ"]
            predicted_outcome = "отказ в удовлетворении"

        risk_score = 0.65
        claims_text = " ".join(case.claims).lower()
        if any(word in claims_text for word in ["неустойк", "штраф", "пеня"]):
            risk_score = 0.82
        if any(word in claims_text for word in ["моральн", "компенсац"]):
            risk_score = 0.75
        if any(word in claims_text for word in ["убытк", "ущерб"]):
            risk_score = 0.70

        recommendations = [
            "Проверить соответствие исковых требований процессуальному законодательству",
            "Запросить экспертизу по представленным документам",
            "Проанализировать судебную практику по аналогичным спорам"
        ]

        contradictions = []
        if len(case.documents) < 2:
            contradictions.append("Недостаточное количество документов для полноценного анализа")
        if "договор" in claims_text and len([d for d in case.documents if "договор" in d.lower()]) == 0:
            contradictions.append("Заявлены требования по договору, но договор не приложен")

        return AnalysisResult(
            similar_cases=similar_cases,
            legal_norms=legal_norms,
            risk_score=risk_score,
            recommendations=recommendations,
            contradictions=contradictions,
            predicted_outcome=predicted_outcome
        )


class ReportPersonalizer:
    @staticmethod
    def personalize(role: UserRole, case: LegalCase, analysis: AnalysisResult) -> PersonalizedReport:
        base_insights = [
            f"Найдено {len(analysis.similar_cases)} аналогичных дел",
            f"Применимые нормы: {', '.join(analysis.legal_norms[:3])}",
            f"Оценка риска: {analysis.risk_score:.0%}",
            f"Прогнозируемый исход: {analysis.predicted_outcome}"
        ]

        actions = []
        warnings = []

        if role == UserRole.JUDGE:
            actions = [
                "Проверить внутренние противоречия в деле",
                f"Сравнить с практикой суда: {analysis.similar_cases[0]['id'] if analysis.similar_cases else 'нет данных'}",
                "Рассчитать объективную сумму компенсации на основе аналогичных дел",
                "Проверить соблюдение процессуальных сроков"
            ]
            warnings = [
                "Внимание: возможен 'эффект якоря' от заявленной суммы иска",
                f"Контроль: {len(analysis.contradictions)} противоречий в документаов",
                "Рекомендуется проверить дело в начале рабочего дня для минимизации влияния усталости"
            ]
        elif role == UserRole.LAWYER:
            actions = [
                f"Использовать аналогичное дело {analysis.similar_cases[0]['id'] if analysis.similar_cases else 'N/A'} в защиту",
                "Подготовить ходатайство об экспертизе",
                "Сформулировать возражения на выявленные противоречия",
                "Подготовить письменные пояснения по слабым местам"
            ]
            warnings = [
                f"Слабые места: {len(analysis.contradictions)} пунктов требуют дополнительной аргументации",
                f"Рекомендуется усилить доказательственную базу",
                f"Вероятность успеха: {100 - analysis.risk_score * 100:.0f}%"
            ]
        elif role == UserRole.PROSECUTOR:
            actions = [
                "Проверить достаточность доказательств для состава преступления",
                "Подготовить проект обвинительного заключения",
                "Сверить сроки давности по делу",
                "Проанализировать позицию защиты"
            ]
            warnings = [
                f"Риск отказа в удовлетворении: {analysis.risk_score:.0%}",
                "Проверить допустимость доказательств",
                "Убедиться в соблюдении процедуры доказывания"
            ]

        return PersonalizedReport(
            role=role,
            generated_at=datetime.now().isoformat(),
            case_id=case.case_id,
            key_insights=base_insights,
            actions=actions,
            warnings=warnings,
            raw_analysis=analysis
        )


class JusticeAIAssistant:
    def __init__(self):
        self.data_manager = DataManager()
        self.core = LegalAnalyticsCore()
        self.personalizer = ReportPersonalizer()

    def create_new_case(self) -> Optional[LegalCase]:
        print("\n" + "=" * 50)
        print("СОЗДАНИЕ НОВОГО ДЕЛА")
        print("=" * 50)
        try:
            case_id = input("Введите номер дела (например, А40-12345/2024): ").strip()
            if not case_id:
                print("Номер дела обязателен!")
                return None
            description = input("Введите описание дела: ").strip()
            case_type = input("Тип дела (арбитражный, гражданский, уголовный, административный): ").strip()
            print("\nВведите стороны (через запятую):")
            parties_input = input("Стороны: ").strip()
            parties = [p.strip() for p in parties_input.split(",") if p.strip()]
            print("\nВведите исковые требования (каждое с новой строки, завершите пустой строкой):")
            claims = []
            while True:
                claim = input("  Требование: ").strip()
                if not claim:
                    break
                claims.append(claim)
            print("\nВведите список документов (через запятую):")
            docs_input = input("Документы: ").strip()
            documents = [d.strip() for d in docs_input.split(",") if d.strip()]
            case = LegalCase(
                case_id=case_id,
                description=description,
                case_type=case_type,
                parties=parties,
                claims=claims,
                documents=documents,
                created_at=datetime.now().isoformat()
            )
            if self.data_manager.save_case(case):
                print(f"Дело '{case_id}' успешно создано и сохранено!")
                return case
            else:
                print("Ошибка при сохранении дела!")
                return None
        except KeyboardInterrupt:
            print("\nСоздание дела прервано")
            return None
        except Exception as e:
            print(f"Ошибка: {e}")
            return None

    def analyze_existing_case(self, case_data: Dict) -> Dict[str, Any]:
        case = LegalCase(**case_data)
        analysis = self.core.analyze_case(case)
        return self._format_analysis(analysis)

    def generate_full_report(self, case: LegalCase, role: UserRole) -> Dict[str, Any]:
        analysis = self.core.analyze_case(case)
        report = self.personalizer.personalize(role, case, analysis)
        saved_path = self.data_manager.save_report(report)
        response = self._format_report(report, saved_path)
        return response

    def _format_analysis(self, analysis: AnalysisResult) -> Dict[str, Any]:
        return {
            "similar_cases": analysis.similar_cases,
            "legal_norms": analysis.legal_norms,
            "risk_score": f"{analysis.risk_score:.0%}",
            "recommendations": analysis.recommendations,
            "contradictions": analysis.contradictions,
            "predicted_outcome": analysis.predicted_outcome
        }

    def _format_report(self, report: PersonalizedReport, saved_path: Optional[str]) -> Dict[str, Any]:
        response = {
            "system": "AI-ассистент правосудия v1.0",
            "user_role": report.role.value,
            "case_id": report.case_id,
            "generated_at": report.generated_at,
            "report": {
                "key_insights": report.key_insights,
                "recommended_actions": report.actions,
                "warnings": report.warnings
            },
            "sources": {
                "similar_cases_count": len(report.raw_analysis.similar_cases),
                "legal_norms": report.raw_analysis.legal_norms[:3]
            },
            "saved_to": saved_path,
            "disclaimer": "Рекомендации носят справочный характер. Ответственность за решение несет пользователь."
        }
        return response


class UserInterface:
    def __init__(self, assistant: JusticeAIAssistant):
        self.assistant = assistant

    def display_main_menu(self):
        while True:
            print("\n" + "=" * 50)
            print("AI-АССИСТЕНТ ДЛЯ ПРАВОСУДИЯ")
            print("=" * 50)
            print("1. Создать новое дело")
            print("2. Проанализировать существующее дело")
            print("3. Просмотреть все сохраненные дела")
            print("4. Сгенерировать отчет для роли")
            print("5. Просмотреть сохраненные отчеты")
            print("6. Выход")
            print("=" * 50)
            choice = input("Выберите действие (1-6): ").strip()
            if choice == "1":
                self.create_case_menu()
            elif choice == "2":
                self.analyze_case_menu()
            elif choice == "3":
                self.view_cases_menu()
            elif choice == "4":
                self.generate_report_menu()
            elif choice == "5":
                self.view_reports_menu()
            elif choice == "6":
                print("\nДо свидания!")
                sys.exit(0)
            else:
                print("Неверный выбор. Попробуйте снова.")

    def create_case_menu(self):
        case = self.assistant.create_new_case()
        if case:
            input("\nНажмите Enter для продолжения...")

    def analyze_case_menu(self):
        print("\n" + "=" * 50)
        print("АНАЛИЗ СУЩЕСТВУЮЩЕГО ДЕЛА")
        print("=" * 50)
        cases = self.assistant.data_manager.load_cases()
        if not cases:
            print("Нет сохраненных дел. Сначала создайте дело.")
            return
        print("\nСохраненные дела:")
        for i, case in enumerate(cases, 1):
            print(f"{i}. {case['case_id']} - {case['description'][:50]}...")
        try:
            choice = int(input("\nВыберите номер дела для анализа: ")) - 1
            if 0 <= choice < len(cases):
                analysis = self.assistant.analyze_existing_case(cases[choice])
                print(f"\nРЕЗУЛЬТАТЫ АНАЛИЗА ДЕЛА {cases[choice]['case_id']}")
                print("=" * 50)
                print(f"Найдено аналогичных дел: {len(analysis['similar_cases'])}")
                print(f"Оценка риска: {analysis['risk_score']}")
                print(f"Прогнозируемый исход: {analysis['predicted_outcome']}")
                print(f"\nРекомендации:")
                for rec in analysis['recommendations']:
                    print(f"  • {rec}")
                if analysis['contradictions']:
                    print(f"\nВыявленные противоречия:")
                    for contr in analysis['contradictions']:
                        print(f"  • {contr}")
            else:
                print("Неверный номер дела.")
        except (ValueError, IndexError):
            print("Неверный выбор.")
        input("\nНажмите Enter для продолжения...")

    def view_cases_menu(self):
        cases = self.assistant.data_manager.load_cases()
        print("\n" + "=" * 50)
        print(f"СОХРАНЕННЫЕ ДЕЛА ({len(cases)})")
        print("=" * 50)
        if not cases:
            print("Нет сохраненных дел.")
        else:
            for i, case in enumerate(cases, 1):
                print(f"\n{i}. Дело: {case['case_id']}")
                print(f"   Описание: {case['description']}")
                print(f"   Тип: {case['case_type']}")
                print(f"   Стороны: {', '.join(case['parties'])}")
                print(f"   Статус: {case.get('status', 'в работе')}")
                print(f"   Создано: {case['created_at'][:10]}")
        input("\nНажмите Enter для продолжения...")

    def generate_report_menu(self):
        cases = self.assistant.data_manager.load_cases()
        if not cases:
            print("Нет сохраненных дел. Сначала создайте дело.")
            return
        print("\n" + "=" * 50)
        print("ГЕНЕРАЦИЯ ОТЧЕТА")
        print("=" * 50)
        print("\nСохраненные дела:")
        for i, case in enumerate(cases, 1):
            print(f"{i}. {case['case_id']}")
        try:
            case_choice = int(input("\nВыберите номер дела: ")) - 1
            if not (0 <= case_choice < len(cases)):
                print("Неверный номер дела.")
                return
            print("\nВыберите роль:")
            roles = list(UserRole)
            for i, role in enumerate(roles, 1):
                print(f"{i}. {role.value}")
            role_choice = int(input("\nВыберите роль (1-3): ")) - 1
            if not (0 <= role_choice < len(roles)):
                print("Неверный номер роли.")
                return
            case_data = cases[case_choice]
            case_obj = LegalCase(**case_data)
            role = roles[role_choice]
            print(f"\nГенерация отчета для {role.value}...")
            report = self.assistant.generate_full_report(case_obj, role)
            print("\n" + "=" * 50)
            print(f"ОТЧЕТ ДЛЯ: {role.value.upper()}")
            print(f"Дело: {report['case_id']}")
            print("=" * 50)
            print("\nКЛЮЧЕВЫЕ ВЫВОДЫ:")
            for insight in report['report']['key_insights']:
                print(f"  • {insight}")
            print("\nРЕКОМЕНДУЕМЫЕ ДЕЙСТВИЯ:")
            for action in report['report']['recommended_actions']:
                print(f"  • {action}")
            print("\nПРЕДУПРЕЖДЕНИЯ:")
            for warning in report['report']['warnings']:
                print(f"  ⚠ {warning}")
            print(f"\nИСТОЧНИКИ:")
            print(f"  Найдено аналогичных дел: {report['sources']['similar_cases_count']}")
            print(f"  Применяемые нормы: {', '.join(report['sources']['legal_norms'])}")
            if report['saved_to']:
                print(f"\nОтчет сохранен в: {report['saved_to']}")
            print(f"\n{report['disclaimer']}")
        except (ValueError, IndexError):
            print("Неверный выбор.")
        input("\nНажмите Enter для продолжения...")

    def view_reports_menu(self):
        reports_dir = self.assistant.data_manager.reports_dir
        print("\n" + "=" * 50)
        print("СОХРАНЕННЫЕ ОТЧЕТЫ")
        print("=" * 50)
        if not os.path.exists(reports_dir):
            print("Директория отчетов не найдена.")
            return
        reports = [f for f in os.listdir(reports_dir) if f.endswith('.json')]
        if not reports:
            print("Нет сохраненных отчетов.")
        else:
            print(f"\nНайдено отчетов: {len(reports)}")
            for report_file in reports[-10:]:
                filepath = os.path.join(reports_dir, report_file)
                try:
                    with open(filepath, 'r', encoding='utf-8') as f:
                        data = json.load(f)
                        role = data.get('role', 'неизвестно')
                        case_id = data.get('case_id', 'неизвестно')
                        date = data.get('generated_at', '')[:10]
                        print(f"  {report_file}")
                        print(f"     Дело: {case_id}, Роль: {role}, Дата: {date}")
                        print()
                except:
                    print(f"  Ошибка чтения: {report_file}")
        input("\nНажмите Enter для продолжения...")


def main():
    print("\n" + "=" * 60)
    print("     AI-АССИСТЕНТ ДЛЯ ПРАВОСУДИЯ - v1.0")
    print("=" * 60)
    assistant = JusticeAIAssistant()
    ui = UserInterface(assistant)
    ui.display_main_menu()


if __name__ == "__main__":
    try:
        main()
    except KeyboardInterrupt:
        print("\nПрограмма завершена пользователем.")
    except Exception as e:
        print(f"\nКритическая ошибка: {e}")
        print("Попробуйте удалить папку 'data' и запустить программу снова.")
