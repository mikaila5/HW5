
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.util.ArrayList;
import java.util.Scanner;
import java.io.PrintWriter;
import java.io.FileOutputStream;
import java.io.FileNotFoundException;
import java.util.Random;

public class RNA {
	private static Scanner inputStream = null;
	private static Helics H;

	public static void main(String[] args) {

		//read  a rna sequence file
		System.out.print("Please enter a file name(eg.DA0260.txt): ");
		inputStream = new Scanner(System.in);
		String fileName = inputStream.nextLine();

		String rnaSeq = readSeq(fileName);

		//Calculate possible pairs and calculate all the possible helics
		ArrayList<Pair> possiblePairs = generatorPair(rnaSeq);
		H = new Helics();
		H = addHelics(possiblePairs, rnaSeq);
		System.out.println(H);
		/*
		String[][] x = readCT("DA0260.ct");
		for (int i = 0; i < checkRow("DA0260.ct"); i++) {
			for (int j = 0; j <= 5; j++) {
				System.out.print(x[i][j] + " ");
			}
			System.out.println();
		}
		//writeCT();
		 * 
		 */
		//RNAtoHelix();
		System.out.println("*********************************");
		//Randomly remix
		helixRandomize(H);
		System.out.println(H);
		cleanUp(H);
		writeCT(helixToRNA(H, rnaSeq));
	}

	/*
	 * This method to read txt file about sequence rna base
	 */
	private static String readSeq(String fileName) {
		try {
			inputStream = new Scanner(new FileInputStream(fileName));

		} catch (FileNotFoundException e) {
			System.out.println("File " + fileName + " was not found");
			System.out.println("or could not be opened.");
			System.exit(0);
		}

		return inputStream.nextLine();
	}

	
	private static int checkRow(String fileName) {
		int count = 0;
		try {
			inputStream = new Scanner(new FileInputStream(fileName));
			inputStream.nextLine();
			while (inputStream.hasNextLine()) {
				inputStream.nextLine();
				count += 1;
			}

		} catch (FileNotFoundException e) {
			System.out.println("File " + fileName + " was not found");
			System.out.println("or could not be opened.");
			System.exit(0);
		}
		return count;
	}

	
	/*
	 * read a ct file and convert to rna structure
	 */
	private static String[][] readCT(String fileName) {
		int row = checkRow(fileName);
		String[][] rnaDataStructure = new String[row][6];
		try {
			inputStream = new Scanner(new FileInputStream(fileName));

			inputStream.nextLine();
			int i = 0;

			while (inputStream.hasNext()) {
				int j = 0;
				while (inputStream.hasNext() && j <= 5) {
					rnaDataStructure[i][j] = inputStream.next();

					j += 1;

				}
				// inputStream.nextLine();
				i += 1;
			}

		} catch (FileNotFoundException e) {
			System.out.println("File " + fileName + " was not found");
			System.out.println("or could not be opened.");
			System.exit(0);
		}

		return rnaDataStructure;
	}

	
	/*
	 * generator all possible pairs from sequence rna data
	 */
	private static ArrayList<Pair> generatorPair(String rndInStr) {
		ArrayList<Pair> possiblePairs = new ArrayList<Pair>();
		for (int i = 1; i <= rndInStr.length() - 1; i++) {
			for (int j = rndInStr.length(); j > i; j--) {
				if (j - i >= 4)
					possiblePairs.add(new Pair(i, j));
			}
		}
		return possiblePairs;
	}

	/*
	 * generator all possible helics
	 */
	private static Helics addHelics(ArrayList<Pair> possiblePairs, String rndSeq) {

		Helix helix = null;
		for (int i = 0; i < possiblePairs.size(); i++) {
			int Ri = possiblePairs.get(i).getI();
			int Rj = possiblePairs.get(i).getJ();
			
			helix = new Helix();
			while (helix.isValid && !helix.isComplete) {
				Pair p = new Pair(Ri, Rj);
				if (p.isCanonicalPair(rndSeq)
						) {
					helix.addBasePair(p);
					Ri++;
					Rj--;
				} else {
					if (helix.isLessThan3BasePairs()) {
						helix.isValid = false;
					} else if (helix.isLessThan3BaseBetweenLastPair()) {
						helix.isValid = false;
					} else {
						helix.isComplete = true;
					}
				}
			}

			if (helix.isValid) {
				H.add(helix);
			}
		}
		return H;
	}

	
	/*
	 * write a rna structure into a ct file
	 */
	public static void writeCT(String[][] rnaStructure) {

		PrintWriter outputStream = null;
		try {
			outputStream = new PrintWriter(
					new FileOutputStream("New DA0260.ct"));

		} catch (FileNotFoundException e) {
			System.out.println("Error file operation");
			System.exit(0);
		}
		System.out.println("Writing to file.");
		outputStream.println(rnaStructure.length+ " ENERGY = 0");
		for (int i = 0; i < rnaStructure.length; i++) {
			for (int j = 0; j <= 5; j++) {
				outputStream.print(rnaStructure[i][j] + " ");
			}
			outputStream.println();
		}

		outputStream.close();
		System.out.println("End of program.");
	}

	/*
	 * convert RNA structure to Helics
	 */
	public static Helics RNAtoHelix() {
		int i = 0;
		String[][] x = readCT("DA0260.ct");
		boolean next = true;
		Helics H = new Helics();
		while (i < checkRow("DA0260.ct")) {
			Helix h = new Helix();
			while (!x[i][4].equals("0")) {
				h.addBasePair(new Pair(i + 1, Integer.parseInt(x[i][4])));
				i += 1;
				next = true;
			}
			if (next) {
				String helix = null;
				H.add(helix);
				next = false;
			}
			i += 1;

		}
		System.out.println(H);
		return H;
	}

	/*
	 * random mix all possible helics
	 */
	public static Helics helixRandomize(Helics H) {
		int randomTime = 0;
		String temp;
		Scanner scan = new Scanner(System.in);
		boolean valid = false;
		while (!valid) {
			try {
				System.out
						.print("How many times do you want to do the random: ");
				randomTime = scan.nextInt();
				if (randomTime > 0) {
					valid = true;
				}
			} catch (Exception e) {
				System.out.print("It must be a positive Integer");
				scan.nextLine();
			}
		}

		Random ran = new Random();
		for (int i = 0; i <= randomTime; i++) {
			int ran1 = ran.nextInt(H.getH().size());
			int ran2 = ran.nextInt(H.getH().size());
			while (ran2 == ran1) {
				ran2 = ran.nextInt(H.getH().size());
			}
			temp = H.getH().get(ran1);
			H.getH().set(ran1, H.getH().get(ran2));
			H.getH().set(ran2, temp);
		}

		scan.close();
		return H;

	}

	/*
	 * clean up for any set that contain conflicting helice
	 */
	public static Helics cleanUp(Helics H) {
		System.out.println("After random\n" + H +"\n//////////////////////");
		int i = 0;
		boolean delete = false;
		while (i< H.getH().size()-1){
			int j = i + 1;
			while (j< H.getH().size()){
				String helix = H.getH().get(i);
				String helix2 = H.getH().get(j);
				for (Pair pair : helix.getBasePairs()){
					if (helix2.alreadyIncluded(pair)){
						delete = true;
						break;
					}
				}
				if (delete){
					H.getH().remove(j);
					delete = false;
				}else{
					j++;
				}
			}
			i++;
		}
		System.out.println("\nAfter cleanup\n" + H);
		return H;

	}
	
	/*
	 * convert helics structure to RNA structure
	 */
	public static String[][] helixToRNA(Helics H, String rnaSeq){
		String[][] rnaStructure = new String[rnaSeq.length()][6];
		for (int i=0; i<rnaSeq.length(); i++){
			rnaStructure[i][0] = Integer.toString(i+1);
			rnaStructure[i][1] = rnaSeq.substring(i, i+1);
			rnaStructure[i][2] = Integer.toString(i);
			rnaStructure[i][3] = Integer.toString(i+2);
			rnaStructure[i][5] = Integer.toString(i+1);
		}
		for (String helix : H.getH()){
			for (Pair pair : helix.getBasePairs()){
				rnaStructure[pair.getI()-1][4] = Integer.toString(pair.getJ());
				rnaStructure[pair.getJ()-1][4] = Integer.toString(pair.getI());
			}
		}
		
		for (int i=0; i<rnaStructure.length; i++){
			if (rnaStructure[i][4] == null){
				rnaStructure[i][4] = "0";
			}
		}
		
		return rnaStructure;
		


	 }
}
