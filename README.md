# JP-
添加，删除
//1
import UIKit

class ViewController: UIViewController {

    
    @IBOutlet weak var myTabelView: UITableView!
    
    var allStudents: [SingleStudent] = []
    
    var session = URLSession(configuration: .default)
    
//    var jsonHost = "http://www.kinwork.jp:7770/LearnApi"
    
    var jsonHost = "https://www.kinwork.jp:1443/LearnApi"
    var oneStudent: SingleStudent!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        myTabelView.delegate = self
        myTabelView.dataSource = self
        //取得所有学生信息
    }
    override func viewWillAppear(_ animated: Bool) {
        super.viewWillAppear(true)
        getAllInfos()
    }

    @IBAction func addBarButtonItrem(_ sender: UIBarButtonItem) {
        performSegue(withIdentifier: "addSegue", sender: nil)
    }
    
    //取得所有人的信息
    func getAllInfos() {
        let urlAddress = "\(jsonHost)/getStudentList"
        if let url = URL(string: urlAddress) {
            let task = session.dataTask(with: url, completionHandler: {
                (data, response, error) in
                if error != nil {
                  print(error!.localizedDescription)
                    return
                }
                if let allInfosData = data {
                    do {
                        self.allStudents = try JSONDecoder().decode([SingleStudent].self, from: allInfosData)
                        DispatchQueue.main.async {
                            self.myTabelView.reloadData()
                        }
                    } catch {
                        print(error.localizedDescription)
                    }
                }
            })
            task.resume()
        }
    }

    
    //根据拿出的学号进行删除
    func deleteInfo(number: String) {
        let URLAddress = "\(jsonHost)/deleteStudent"
        if let url = URL(string: URLAddress) {
            let para: [String:String] = ["s_no":"\(number)"]
            let postData = editData(para: para)
            var request = URLRequest(url: url)
            request.httpMethod = "POST"
            request.httpBody = postData
            let task = session.dataTask(with: request, completionHandler: {
                (data, response, error) in
                if error != nil {
                    print(error!.localizedDescription)
                    return
                }
                if let deleteData = data {
                    do {
                        let jsonResult = try JSONSerialization.jsonObject(with: deleteData, options: []) as? NSDictionary
                        if let dictResult = jsonResult {
                            if let boolResult = dictResult["deleteResult"] as? Bool {
                                if boolResult {
                                    self.getAllInfos()
                                }
                            }
                        }
                    } catch {
                        print(error.localizedDescription)
                    }
                }
            })
            task.resume()
        }
    }
    //取得单一学生信息的方法
    func getSingleInfo(number: String) {
        let URLAddress = "\(jsonHost)/viewStudent"
        if let url = URL(string: URLAddress) {
            let para: [String:String] = ["s_no":"\(number)"]
            let postData = editData(para: para)

            var request = URLRequest(url: url)
            request.httpMethod = "POST"
            request.httpBody = postData
    
            let task = session.dataTask(with: request, completionHandler: {
                (data, response, error) in
                if error != nil {
                    print(error!.localizedDescription)
                }
                if let singleInfoData = data {
                    do {
                        self.oneStudent = try JSONDecoder().decode(SingleStudent.self, from: singleInfoData)
                        DispatchQueue.main.async {
                            self.performSegue(withIdentifier: "nextSegue", sender: nil)
                        }
                    } catch {
                        print(error.localizedDescription)
                    }
                }
            })
            task.resume()
        }
    }
    
    
  
    func editData(para: [String: String]) -> Data? {
        let list = NSMutableArray()
        if para.count > 0 {
            for (key,value) in para {
                let temp = "\(key)=\(value)"
                list.add(temp)
            }
        }
        let strPara = list.componentsJoined(by: "&")
        let dataPara  = strPara.data(using: String.Encoding.utf8)
        return dataPara
    }
    
    override func prepare(for segue: UIStoryboardSegue, sender: Any?) {
        if segue.identifier == "nextSegue" {
            let nextPage = segue.destination as! SingleInfoViewController
            nextPage.singeleStudent = self.oneStudent
        }
    }
}

extension ViewController: UITableViewDelegate {
    func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
        if let studentNo = allStudents[indexPath.row].s_no {
            getSingleInfo(number: studentNo)
        }
    }
    
    func tableView(_ tableView: UITableView, editingStyleForRowAt indexPath: IndexPath) -> UITableViewCellEditingStyle {
        return .delete
    }
    func tableView(_ tableView: UITableView, titleForDeleteConfirmationButtonForRowAt indexPath: IndexPath) -> String? {
        return "删除"
    }
    func tableView(_ tableView: UITableView, commit editingStyle: UITableViewCellEditingStyle, forRowAt indexPath: IndexPath) {
        if let studentNo = allStudents[indexPath.row].s_no {
            deleteInfo(number: studentNo)
        }
    }
}
extension ViewController: UITableViewDataSource {
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return allStudents.count
    }
    
    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let cell = tableView.dequeueReusableCell(withIdentifier: "infoCell", for: indexPath) as! InFoTableViewCell
        cell.nameLabel.text = allStudents[indexPath.row].name!
        cell.birthdayLabel.text = allStudents[indexPath.row].birthday!
        return cell
    }
    func tableView(_ tableView: UITableView, heightForRowAt indexPath: IndexPath) -> CGFloat {
        return 80
    }
}

//2
import UIKit

class InFoTableViewCell: UITableViewCell {

    
    @IBOutlet weak var nameLabel: UILabel!
    
    @IBOutlet weak var birthdayLabel: UILabel!
    
    
    override func awakeFromNib() {
        super.awakeFromNib()
        
        self.selectionStyle = .none
    }

}
//3
import UIKit

class SingleInfoViewController: UIViewController {

    
    @IBOutlet weak var infoLabel: UILabel!
    
    
    var singeleStudent: SingleStudent?
    
    
    override func viewDidLoad() {
        super.viewDidLoad()
    
        setupView()
    }
    func setupView() {
        var text = ""
        if let info = singeleStudent {
            text.append("姓名:\(info.name!)\n")
            if let grade = info.grade {
                text.append("年级:\(grade)\n")
            }
            text.append("生日:\(info.birthday!)\n")
            if let phone = info.phone_number {
                text.append("电话:\(phone)\n")
            }
        }
        infoLabel.text = text
    }
}

//4
import UIKit

class AddInfoViewController: UIViewController {

    
    @IBOutlet weak var nameTextField: UITextField!
    
    @IBOutlet weak var gradeTextField: UITextField!
    
    @IBOutlet weak var birthdayTextField: UITextField!
    
    @IBOutlet weak var phoneTextField: UITextField!
    
 
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
    }
    
    @IBAction func onlyBackButton(_ sender: UIButton) {
        dismiss(animated: true, completion: nil)
    }
    

    func popAlert(title: String,actionTitle: String,message: String) {
        let alertView = UIAlertController(title: title, message: message, preferredStyle: .alert)
        let actionView = UIAlertAction(title: actionTitle, style: .default, handler: nil)
        alertView.addAction(actionView)
        DispatchQueue.main.async {
            self.present(alertView,animated: true,completion: nil)
        }
    }
    
    @IBAction func backButton(_ sender: UIButton) {
        if nameTextField.text! == "" {
            popAlert(title: "温馨提示", actionTitle: "确认", message: "亲：名字不能为空哦")
        }
        if birthdayTextField.text! == "" {
            popAlert(title: "温馨提示", actionTitle: "OK", message: "亲：生日不能为空哦")
        }
        
        let URLAddress = "\(jsonHost)/addStudent"
        if let url = URL(string: URLAddress) {
            let name = nameTextField.text!
            let birthday = birthdayTextField.text!
            var para: [String:String] = ["name":"\(name)","birthday":"\(birthday)"]
            
            if gradeTextField.text != "" {
                let grade = gradeTextField.text!
                para["grade"] = grade
            }
            if phoneTextField.text != "" {
                let phone = phoneTextField.text!
                para["phone_number"] = phone
            }
            
            let postData = editData(para: para)
            
            var request = URLRequest(url: url)
            request.httpMethod = "POST"
            request.httpBody = postData
            
            let task = session.dataTask(with: request, completionHandler: {
                (data, response, error) in
                if error != nil {
                    self.popAlert(title: "Error", actionTitle: "Ok", message: "Request Error")
                    return
                }
                if let createInfoData = data {
                    do {
                        let jsonResult = try JSONSerialization.jsonObject(with: createInfoData, options: []) as? [String : String]
                        if let result = jsonResult {
                            print(result)
                            if result["message"] == "处理成功" {
                                //                                print("true")
                                DispatchQueue.main.async {
                                    self.dismiss(animated: true, completion: nil)
                                }
                            }
                        }
                    } catch {
                        print(error.localizedDescription)
                    }
                }
            })
            task.resume()
        }
    }
}

//5
import UIKit

class RemoveViewController: UIViewController {

    
    
    @IBOutlet weak var numberTextField: UITextField!
    
    @IBOutlet weak var cutButton: UIButton!
    
    override func viewDidLoad() {
        super.viewDidLoad()

        cutButton.layer.cornerRadius = cutButton.frame.height / 2
        cutButton.layer.masksToBounds = true
    }

    func deleteInfo(number: String) {
        let URLAddress = "\(jsonHost)/deleteStudent"
        if let url = URL(string: URLAddress) {
            let para: [String:String] = ["s_no":"\(number)"]
            let postData = editData(para: para)
            var request = URLRequest(url: url)
            request.httpMethod = "POST"
            request.httpBody = postData
            let task = session.dataTask(with: request, completionHandler: {
                (data, response, error) in
                if error != nil {
                 print(error!.localizedDescription)
                    return
                }
                if let deleteData = data {
                    do {
                        let jsonResult = try JSONSerialization.jsonObject(with: deleteData, options: []) as? [String: String]
                        if let result = jsonResult {
                            print(result)
                            if result["deleteResult"] == "true" {
                                self.dismiss(animated: true, completion: nil)
                            }
                        }
                    } catch {
                        print(error.localizedDescription)
                    }
                }
            })
            task.resume()
        }
    }
    
    @IBAction func removeInfoButton(_ sender: UIButton) {
        if numberTextField.text != "" {
             deleteInfo(number: numberTextField.text!)
            } else {
            
        }
    }
    
    
}
//6
import Foundation

struct SingleStudent: Decodable {
    var s_no: String?
    var name: String?
    var grade: String?
    var birthday: String?
    var phone_number: String?
}
//7
import Foundation

var jsonHost = "https://www.kinwork.jp:1443/LearnApi"
//var jsonHost = "http://www.kinwork.jp:7770/LearnApi"

var session = URLSession(configuration: .default)

func editData(para: [String: String]) -> Data? {
    let list = NSMutableArray()
    if para.count > 0 {
        for (key,value) in para {
            let temp = "\(key)=\(value)"
            list.add(temp)
        }
    }
    let strPara = list.componentsJoined(by: "&")
    let dataPara  = strPara.data(using: String.Encoding.utf8)
    return dataPara
}

             

                
                
                    
 
